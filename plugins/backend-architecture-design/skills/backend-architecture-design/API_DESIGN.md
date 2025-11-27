# API Design Best Practices for FastAPI

This guide covers best practices for designing RESTful APIs with FastAPI, focusing on Pydantic models, dependency injection, routing, and OpenAPI documentation.

## 1. RESTful API Design Patterns

### Resource Naming Conventions

**Good Examples:**
```python
# Use plural nouns for collections
GET    /api/v1/users
POST   /api/v1/users
GET    /api/v1/users/{user_id}
PUT    /api/v1/users/{user_id}
DELETE /api/v1/users/{user_id}

# Nested resources for relationships
GET    /api/v1/users/{user_id}/posts
POST   /api/v1/users/{user_id}/posts
GET    /api/v1/users/{user_id}/posts/{post_id}
```

**Bad Examples:**
```python
# ❌ Using verbs in URLs
POST   /api/v1/createUser
GET    /api/v1/getUserById/{id}

# ❌ Inconsistent naming
GET    /api/v1/user          # Should be plural
POST   /api/v1/users/create  # Redundant verb
```

### HTTP Methods and Status Codes

**Proper Usage:**
```python
from fastapi import FastAPI, HTTPException, status
from typing import List

@app.post("/users", status_code=status.HTTP_201_CREATED)
async def create_user(user: UserCreate) -> UserResponse:
    """Create a new user - returns 201 Created"""
    return created_user

@app.get("/users/{user_id}", status_code=status.HTTP_200_OK)
async def get_user(user_id: int) -> UserResponse:
    """Get user by ID - returns 200 OK or 404 Not Found"""
    user = await get_user_from_db(user_id)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"User {user_id} not found"
        )
    return user

@app.put("/users/{user_id}", status_code=status.HTTP_200_OK)
async def update_user(user_id: int, user: UserUpdate) -> UserResponse:
    """Full update - returns 200 OK"""
    return updated_user

@app.patch("/users/{user_id}", status_code=status.HTTP_200_OK)
async def partial_update_user(user_id: int, user: UserPartialUpdate) -> UserResponse:
    """Partial update - returns 200 OK"""
    return updated_user

@app.delete("/users/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_user(user_id: int):
    """Delete user - returns 204 No Content"""
    await delete_user_from_db(user_id)
```

## 2. Pydantic Model Design

### Separate Models for Different Purposes

**Best Practice: Create Distinct Models**
```python
from pydantic import BaseModel, EmailStr, Field, ConfigDict
from datetime import datetime
from typing import Optional

# Base model with common fields
class UserBase(BaseModel):
    email: EmailStr
    username: str = Field(..., min_length=3, max_length=50)
    full_name: Optional[str] = None

# Request model for creation (no ID, no timestamps)
class UserCreate(UserBase):
    password: str = Field(..., min_length=8)

# Request model for updates (all fields optional)
class UserUpdate(BaseModel):
    email: Optional[EmailStr] = None
    username: Optional[str] = Field(None, min_length=3, max_length=50)
    full_name: Optional[str] = None

# Response model (includes ID, no password, includes timestamps)
class UserResponse(UserBase):
    id: int
    is_active: bool
    created_at: datetime
    updated_at: datetime

    model_config = ConfigDict(from_attributes=True)

# Internal model for database operations (includes password hash)
class UserInDB(UserResponse):
    hashed_password: str
```

**Anti-pattern: Single Model for Everything**
```python
# ❌ Bad: One model used for everything
class User(BaseModel):
    id: Optional[int] = None           # Optional for creation
    email: EmailStr
    password: Optional[str] = None     # Exposed in responses!
    hashed_password: Optional[str] = None
    created_at: Optional[datetime] = None

    model_config = ConfigDict(from_attributes=True)
```

### Pydantic Validators and Field Constraints

**Good Examples:**
```python
from pydantic import BaseModel, Field, field_validator, model_validator
import re

class UserCreate(BaseModel):
    username: str = Field(
        ...,
        min_length=3,
        max_length=50,
        pattern="^[a-zA-Z0-9_-]+$",
        description="Username can only contain letters, numbers, underscores, and hyphens"
    )
    email: EmailStr
    password: str = Field(..., min_length=8)
    age: int = Field(..., ge=18, le=150, description="User must be 18 or older")

    @field_validator('password')
    @classmethod
    def validate_password(cls, v: str) -> str:
        if not re.search(r'[A-Z]', v):
            raise ValueError('Password must contain at least one uppercase letter')
        if not re.search(r'[a-z]', v):
            raise ValueError('Password must contain at least one lowercase letter')
        if not re.search(r'\d', v):
            raise ValueError('Password must contain at least one digit')
        return v

    @field_validator('username')
    @classmethod
    def validate_username(cls, v: str) -> str:
        if v.lower() in ['admin', 'root', 'system']:
            raise ValueError('This username is reserved')
        return v

class DateRangeQuery(BaseModel):
    start_date: datetime
    end_date: datetime

    @model_validator(mode='after')
    def validate_date_range(self):
        if self.start_date >= self.end_date:
            raise ValueError('start_date must be before end_date')
        return self
```

## 3. Dependency Injection

### Database Session Dependency

**Good Example:**
```python
from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession
from typing import AsyncGenerator

async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with async_session_maker() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()

# Usage in routes
@app.get("/users/{user_id}")
async def get_user(
    user_id: int,
    db: AsyncSession = Depends(get_db)
) -> UserResponse:
    user = await db.get(User, user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

### Authentication Dependency

**Good Example:**
```python
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from jose import JWTError, jwt

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

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
        user_id: int = payload.get("sub")
        if user_id is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception

    user = await db.get(User, user_id)
    if user is None:
        raise credentials_exception
    return user

async def get_current_active_user(
    current_user: User = Depends(get_current_user)
) -> User:
    if not current_user.is_active:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user

# Usage in routes
@app.get("/users/me")
async def read_users_me(
    current_user: User = Depends(get_current_active_user)
) -> UserResponse:
    return current_user
```

### Configuration Dependency

**Good Example:**
```python
from functools import lru_cache
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    secret_key: str
    algorithm: str = "HS256"
    access_token_expire_minutes: int = 30

    class Config:
        env_file = ".env"

@lru_cache()
def get_settings() -> Settings:
    return Settings()

# Usage in routes
@app.get("/info")
async def get_info(settings: Settings = Depends(get_settings)):
    return {"app_name": "My API", "environment": settings.environment}
```

### Service Layer Dependency

**Good Example:**
```python
class UserService:
    def __init__(self, db: AsyncSession):
        self.db = db

    async def get_user(self, user_id: int) -> Optional[User]:
        return await self.db.get(User, user_id)

    async def create_user(self, user_data: UserCreate) -> User:
        # Business logic here
        hashed_password = hash_password(user_data.password)
        db_user = User(
            **user_data.model_dump(exclude={'password'}),
            hashed_password=hashed_password
        )
        self.db.add(db_user)
        await self.db.flush()
        await self.db.refresh(db_user)
        return db_user

def get_user_service(db: AsyncSession = Depends(get_db)) -> UserService:
    return UserService(db)

# Usage in routes
@app.post("/users", status_code=status.HTTP_201_CREATED)
async def create_user(
    user: UserCreate,
    user_service: UserService = Depends(get_user_service)
) -> UserResponse:
    return await user_service.create_user(user)
```

## 4. Router Organization

### Modular Router Structure

**Good Example:**
```python
# app/routers/users.py
from fastapi import APIRouter, Depends, HTTPException, status

router = APIRouter(
    prefix="/users",
    tags=["users"],
    responses={404: {"description": "Not found"}},
)

@router.get("/", response_model=List[UserResponse])
async def list_users(
    skip: int = 0,
    limit: int = 100,
    db: AsyncSession = Depends(get_db)
):
    users = await get_users(db, skip=skip, limit=limit)
    return users

@router.post("/", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def create_user(user: UserCreate, db: AsyncSession = Depends(get_db)):
    return await create_user_in_db(db, user)

# app/main.py
from fastapi import FastAPI
from app.routers import users, posts, comments

app = FastAPI(title="My API", version="1.0.0")

app.include_router(users.router, prefix="/api/v1")
app.include_router(posts.router, prefix="/api/v1")
app.include_router(comments.router, prefix="/api/v1")
```

## 5. API Versioning

### URL-based Versioning

**Good Example:**
```python
# Explicit version in URL prefix
v1_router = APIRouter(prefix="/api/v1")
v2_router = APIRouter(prefix="/api/v2")

# app/routers/v1/users.py
router = APIRouter(prefix="/users", tags=["users-v1"])

# app/routers/v2/users.py
router = APIRouter(prefix="/users", tags=["users-v2"])

# main.py
app.include_router(v1_router)
app.include_router(v2_router)
```

## 6. OpenAPI Documentation

### Enhanced Documentation with Examples

**Good Example:**
```python
from pydantic import BaseModel, Field

class UserCreate(BaseModel):
    username: str = Field(
        ...,
        min_length=3,
        max_length=50,
        description="Unique username for the user",
        examples=["john_doe"]
    )
    email: EmailStr = Field(
        ...,
        description="User's email address",
        examples=["john@example.com"]
    )
    full_name: str = Field(
        ...,
        description="User's full name",
        examples=["John Doe"]
    )

@app.post(
    "/users",
    response_model=UserResponse,
    status_code=status.HTTP_201_CREATED,
    summary="Create a new user",
    description="Create a new user with the provided information. Username must be unique.",
    response_description="The created user",
    responses={
        201: {
            "description": "User successfully created",
            "content": {
                "application/json": {
                    "example": {
                        "id": 1,
                        "username": "john_doe",
                        "email": "john@example.com",
                        "full_name": "John Doe",
                        "is_active": True,
                        "created_at": "2024-01-01T00:00:00"
                    }
                }
            }
        },
        400: {"description": "Invalid input data"},
        409: {"description": "Username or email already exists"}
    },
    tags=["users"]
)
async def create_user(user: UserCreate) -> UserResponse:
    """
    Create a new user with all the information:

    - **username**: Unique identifier (3-50 characters)
    - **email**: Valid email address
    - **full_name**: User's full name
    """
    return await create_user_in_db(user)
```

## 7. Error Response Standardization

### Consistent Error Format

**Good Example:**
```python
from fastapi import HTTPException
from pydantic import BaseModel
from typing import Optional, Dict, Any

class ErrorResponse(BaseModel):
    error: str
    message: str
    details: Optional[Dict[str, Any]] = None

class AppException(HTTPException):
    def __init__(
        self,
        status_code: int,
        error: str,
        message: str,
        details: Optional[Dict[str, Any]] = None
    ):
        super().__init__(
            status_code=status_code,
            detail={
                "error": error,
                "message": message,
                "details": details
            }
        )

# Usage
raise AppException(
    status_code=404,
    error="USER_NOT_FOUND",
    message="User with the specified ID does not exist",
    details={"user_id": user_id}
)

# Global exception handler
from fastapi import Request
from fastapi.responses import JSONResponse

@app.exception_handler(AppException)
async def app_exception_handler(request: Request, exc: AppException):
    return JSONResponse(
        status_code=exc.status_code,
        content=exc.detail
    )
```

## Review Checklist

When reviewing API design, check for:

- [ ] RESTful naming conventions (plural nouns, no verbs in URLs)
- [ ] Appropriate HTTP methods and status codes
- [ ] Separate Pydantic models for Request/Response/Internal use
- [ ] Proper field validation and constraints
- [ ] Dependency injection for database sessions, services, and configuration
- [ ] Modular router organization
- [ ] API versioning strategy
- [ ] Comprehensive OpenAPI documentation with examples
- [ ] Consistent error response format
- [ ] Proper use of response_model to avoid data leakage
- [ ] Tags for API documentation organization

## Common Issues to Look For

1. **Password in Response Models** - Use Grep to find `password` in response models
2. **Missing Status Codes** - Routes without explicit `status_code` parameter
3. **Global Variables** - Database sessions or services created globally instead of using dependencies
4. **Poor Error Messages** - Generic error messages without helpful details
5. **Missing Validators** - Pydantic models without proper field validation
6. **Inconsistent Naming** - Mixed singular/plural resource names
7. **Missing Documentation** - Routes without descriptions or examples
