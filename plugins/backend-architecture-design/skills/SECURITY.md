# Security Best Practices for FastAPI

This guide covers authentication, authorization, data validation, CORS, SQL injection prevention, and sensitive data handling for FastAPI applications.

## 1. Authentication

### OAuth2 with JWT Tokens

**Good Example:**
```python
from datetime import datetime, timedelta
from typing import Optional
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel

# Configuration
SECRET_KEY = "your-secret-key-here"  # Should be in environment variables
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    username: Optional[str] = None

# Password hashing
def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)

# JWT token creation
def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

# Authenticate user
async def authenticate_user(db: AsyncSession, username: str, password: str):
    user = await get_user_by_username(db, username)
    if not user:
        return False
    if not verify_password(password, user.hashed_password):
        return False
    return user

# Get current user from token
async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: AsyncSession = Depends(get_db)
) -> User:
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
        token_data = TokenData(username=username)
    except JWTError:
        raise credentials_exception

    user = await get_user_by_username(db, username=token_data.username)
    if user is None:
        raise credentials_exception
    return user

async def get_current_active_user(
    current_user: User = Depends(get_current_user)
) -> User:
    if not current_user.is_active:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user

# Login endpoint
@app.post("/token", response_model=Token)
async def login(
    form_data: OAuth2PasswordRequestForm = Depends(),
    db: AsyncSession = Depends(get_db)
):
    user = await authenticate_user(db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}

# Protected endpoint
@app.get("/users/me", response_model=UserResponse)
async def read_users_me(current_user: User = Depends(get_current_active_user)):
    return current_user
```

### Refresh Tokens

**Good Example:**
```python
from uuid import uuid4

REFRESH_TOKEN_EXPIRE_DAYS = 7

class RefreshToken(Base):
    __tablename__ = "refresh_tokens"

    id: Mapped[int] = mapped_column(primary_key=True)
    token: Mapped[str] = mapped_column(String(255), unique=True, index=True)
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"))
    expires_at: Mapped[datetime] = mapped_column(DateTime(timezone=True))
    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now()
    )

    user: Mapped["User"] = relationship("User")

def create_refresh_token(user_id: int) -> str:
    return f"{user_id}.{uuid4().hex}"

@app.post("/token/refresh")
async def refresh_access_token(
    refresh_token: str,
    db: AsyncSession = Depends(get_db)
):
    # Verify refresh token
    stmt = select(RefreshToken).where(
        RefreshToken.token == refresh_token,
        RefreshToken.expires_at > datetime.utcnow()
    )
    result = await db.execute(stmt)
    db_token = result.scalar_one_or_none()

    if not db_token:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or expired refresh token"
        )

    # Create new access token
    access_token = create_access_token(
        data={"sub": db_token.user.username}
    )

    return {"access_token": access_token, "token_type": "bearer"}
```

## 2. Authorization

### Role-Based Access Control (RBAC)

**Good Example:**
```python
from enum import Enum
from typing import List

class UserRole(str, Enum):
    ADMIN = "admin"
    MODERATOR = "moderator"
    USER = "user"

class User(Base):
    __tablename__ = "users"
    # ... other fields ...
    role: Mapped[str] = mapped_column(String(20), default=UserRole.USER)

# Permission checker
class RoleChecker:
    def __init__(self, allowed_roles: List[UserRole]):
        self.allowed_roles = allowed_roles

    def __call__(self, current_user: User = Depends(get_current_active_user)):
        if current_user.role not in self.allowed_roles:
            raise HTTPException(
                status_code=status.HTTP_403_FORBIDDEN,
                detail="Operation not permitted"
            )
        return current_user

# Usage in routes
allow_admin = RoleChecker([UserRole.ADMIN])
allow_admin_or_moderator = RoleChecker([UserRole.ADMIN, UserRole.MODERATOR])

@app.delete("/users/{user_id}")
async def delete_user(
    user_id: int,
    current_user: User = Depends(allow_admin),
    db: AsyncSession = Depends(get_db)
):
    # Only admins can access this endpoint
    await delete_user_from_db(db, user_id)
    return {"status": "deleted"}

@app.patch("/posts/{post_id}/moderate")
async def moderate_post(
    post_id: int,
    current_user: User = Depends(allow_admin_or_moderator),
    db: AsyncSession = Depends(get_db)
):
    # Admins and moderators can access this
    pass
```

### Resource-Based Authorization

**Good Example:**
```python
async def verify_post_owner(
    post_id: int,
    current_user: User = Depends(get_current_active_user),
    db: AsyncSession = Depends(get_db)
) -> Post:
    """Verify that current user owns the post"""
    post = await db.get(Post, post_id)
    if not post:
        raise HTTPException(status_code=404, detail="Post not found")

    if post.author_id != current_user.id and current_user.role != UserRole.ADMIN:
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Not authorized to modify this post"
        )

    return post

@app.put("/posts/{post_id}")
async def update_post(
    post_id: int,
    post_update: PostUpdate,
    post: Post = Depends(verify_post_owner),
    db: AsyncSession = Depends(get_db)
):
    # User owns the post or is admin
    for key, value in post_update.model_dump(exclude_unset=True).items():
        setattr(post, key, value)
    await db.commit()
    return post
```

## 3. Input Validation and Sanitization

### Pydantic Validation

**Good Example:**
```python
from pydantic import BaseModel, Field, field_validator, EmailStr
import re
from typing import Optional

class UserCreate(BaseModel):
    email: EmailStr  # Validates email format
    username: str = Field(
        ...,
        min_length=3,
        max_length=50,
        pattern="^[a-zA-Z0-9_-]+$"
    )
    password: str = Field(..., min_length=8, max_length=100)
    age: Optional[int] = Field(None, ge=13, le=150)

    @field_validator('password')
    @classmethod
    def validate_password_strength(cls, v: str) -> str:
        if not re.search(r'[A-Z]', v):
            raise ValueError('Password must contain uppercase letter')
        if not re.search(r'[a-z]', v):
            raise ValueError('Password must contain lowercase letter')
        if not re.search(r'\d', v):
            raise ValueError('Password must contain digit')
        if not re.search(r'[!@#$%^&*(),.?":{}|<>]', v):
            raise ValueError('Password must contain special character')
        return v

    @field_validator('username')
    @classmethod
    def validate_username(cls, v: str) -> str:
        # Prevent SQL injection in username
        forbidden = ['--', ';', '/*', '*/', 'xp_', 'sp_', 'DROP', 'SELECT']
        v_upper = v.upper()
        if any(word in v_upper for word in forbidden):
            raise ValueError('Username contains forbidden characters')
        return v
```

### SQL Injection Prevention

**Good Example (Safe):**
```python
from sqlalchemy import select, text

# ✅ Good: Using SQLAlchemy ORM (parameters are automatically escaped)
@app.get("/users/search")
async def search_users(
    query: str,
    db: AsyncSession = Depends(get_db)
):
    stmt = select(User).where(User.username.ilike(f"%{query}%"))
    result = await db.execute(stmt)
    return result.scalars().all()

# ✅ Good: Using bound parameters with raw SQL
@app.get("/users/by-email")
async def get_user_by_email(
    email: str,
    db: AsyncSession = Depends(get_db)
):
    stmt = text("SELECT * FROM users WHERE email = :email")
    result = await db.execute(stmt, {"email": email})
    return result.first()
```

**Anti-pattern (Vulnerable):**
```python
# ❌ NEVER DO THIS: String concatenation with user input
@app.get("/users/search")
async def search_users_unsafe(
    query: str,
    db: AsyncSession = Depends(get_db)
):
    # VULNERABLE TO SQL INJECTION!
    stmt = text(f"SELECT * FROM users WHERE username LIKE '%{query}%'")
    result = await db.execute(stmt)
    return result.all()
```

## 4. CORS Configuration

**Good Example:**
```python
from fastapi.middleware.cors import CORSMiddleware

# Development configuration
if settings.ENVIRONMENT == "development":
    origins = ["http://localhost:3000", "http://localhost:8080"]
else:
    # Production: specific allowed origins
    origins = [
        "https://yourapp.com",
        "https://www.yourapp.com",
    ]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,  # Don't use ["*"] in production
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE", "PATCH"],  # Specify allowed methods
    allow_headers=["*"],  # Or specify: ["Content-Type", "Authorization"]
    max_age=600,  # Cache preflight requests for 10 minutes
)
```

**Anti-pattern:**
```python
# ❌ Bad: Allowing all origins in production
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # INSECURE!
    allow_credentials=True,  # Dangerous with allow_origins=["*"]
    allow_methods=["*"],
    allow_headers=["*"],
)
```

## 5. Sensitive Data Handling

### Environment Variables

**Good Example:**
```python
from pydantic_settings import BaseSettings
from functools import lru_cache

class Settings(BaseSettings):
    # Database
    database_url: str

    # Security
    secret_key: str
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 30

    # API Keys (never hardcode!)
    sendgrid_api_key: str
    stripe_secret_key: str

    # Environment
    environment: str = "development"
    debug: bool = False

    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"

@lru_cache()
def get_settings() -> Settings:
    return Settings()

# Usage
settings = get_settings()

# ❌ NEVER commit .env file to git!
# ✅ Include .env in .gitignore
# ✅ Provide .env.example with dummy values
```

### Password Storage

**Good Example:**
```python
from passlib.context import CryptContext

# Use bcrypt for password hashing
pwd_context = CryptContext(
    schemes=["bcrypt"],
    deprecated="auto",
    bcrypt__rounds=12  # Higher is more secure but slower
)

def hash_password(password: str) -> str:
    return pwd_context.hash(password)

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

# ❌ NEVER store plain text passwords
# ❌ NEVER use weak hashing like MD5 or SHA1
# ✅ Use bcrypt, argon2, or scrypt
```

### Sensitive Data in Responses

**Good Example:**
```python
from pydantic import BaseModel, ConfigDict

class UserResponse(BaseModel):
    id: int
    email: str
    username: str
    is_active: bool

    # ❌ DON'T include: password, hashed_password, api_keys, tokens

    model_config = ConfigDict(from_attributes=True)

@app.get("/users/{user_id}", response_model=UserResponse)
async def get_user(user_id: int, db: AsyncSession = Depends(get_db)):
    user = await db.get(User, user_id)
    # Pydantic automatically excludes fields not in UserResponse
    return user
```

## 6. Rate Limiting

**Good Example:**
```python
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

# Apply to specific routes
@app.post("/login")
@limiter.limit("5/minute")  # Max 5 login attempts per minute
async def login(
    request: Request,
    form_data: OAuth2PasswordRequestForm = Depends()
):
    # Login logic
    pass

@app.post("/api/data")
@limiter.limit("100/hour")
async def create_data(request: Request, data: DataCreate):
    # Data creation logic
    pass
```

## 7. Request Validation and Security Headers

**Good Example:**
```python
from fastapi import Request
from fastapi.responses import JSONResponse
import secrets

# CSRF Protection
@app.middleware("http")
async def add_security_headers(request: Request, call_next):
    response = await call_next(request)

    # Security headers
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    response.headers["X-XSS-Protection"] = "1; mode=block"
    response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"

    # Generate CSP nonce for inline scripts (if needed)
    nonce = secrets.token_urlsafe(16)
    response.headers["Content-Security-Policy"] = (
        f"default-src 'self'; "
        f"script-src 'self' 'nonce-{nonce}'; "
        f"style-src 'self' 'unsafe-inline'; "
        f"img-src 'self' data: https:;"
    )

    return response

# Request size limiting
@app.middleware("http")
async def limit_upload_size(request: Request, call_next):
    if request.method in ["POST", "PUT", "PATCH"]:
        content_length = request.headers.get("content-length")
        if content_length:
            content_length = int(content_length)
            max_size = 10 * 1024 * 1024  # 10 MB
            if content_length > max_size:
                return JSONResponse(
                    status_code=413,
                    content={"detail": "Request entity too large"}
                )

    return await call_next(request)
```

## 8. Logging and Monitoring (Security)

**Good Example:**
```python
import logging
from fastapi import Request

logger = logging.getLogger(__name__)

@app.middleware("http")
async def log_requests(request: Request, call_next):
    # Log request (but don't log sensitive data!)
    logger.info(
        f"Request: {request.method} {request.url.path} "
        f"from {request.client.host}"
    )

    response = await call_next(request)

    # Log failed authentication attempts
    if response.status_code == 401:
        logger.warning(
            f"Unauthorized access attempt: {request.method} {request.url.path} "
            f"from {request.client.host}"
        )

    return response

# ❌ NEVER log: passwords, tokens, API keys, credit card numbers
# ✅ Log: authentication failures, authorization failures, suspicious patterns
```

## Review Checklist

When reviewing security, check for:

- [ ] JWT tokens with expiration
- [ ] Refresh token mechanism implemented
- [ ] Passwords hashed with bcrypt/argon2
- [ ] Role-based or resource-based authorization
- [ ] Input validation on all user inputs
- [ ] SQL injection prevention (using ORM or bound parameters)
- [ ] CORS properly configured (no wildcard in production)
- [ ] Sensitive data not exposed in API responses
- [ ] Environment variables for secrets (not hardcoded)
- [ ] Rate limiting on authentication endpoints
- [ ] Security headers configured
- [ ] Request size limits
- [ ] HTTPS enforced in production
- [ ] Proper error messages (don't leak implementation details)

## Common Security Issues to Look For

1. **Hardcoded Secrets** - Use Grep to find API keys, passwords in code
2. **Password in Responses** - Check Pydantic response models
3. **SQL Injection** - Find raw SQL with string concatenation
4. **Missing Authentication** - Routes without `Depends(get_current_user)`
5. **Weak Password Rules** - No password strength validation
6. **CORS Misconfiguration** - `allow_origins=["*"]` with credentials
7. **No Rate Limiting** - Login/signup endpoints without rate limits
8. **Sensitive Data Logging** - Passwords or tokens in log statements
