# Code Organization Best Practices for FastAPI

This guide covers project structure, layered architecture, dependency injection, configuration management, and code modularity for FastAPI applications.

## 1. Project Structure

### Recommended Directory Layout

**Good Example:**
```
project_root/
├── app/
│   ├── __init__.py
│   ├── main.py                 # FastAPI app instance and startup
│   ├── config.py               # Configuration and settings
│   ├── dependencies.py         # Shared dependencies
│   │
│   ├── api/                    # API layer
│   │   ├── __init__.py
│   │   ├── v1/                 # API version 1
│   │   │   ├── __init__.py
│   │   │   ├── endpoints/      # Route handlers
│   │   │   │   ├── __init__.py
│   │   │   │   ├── users.py
│   │   │   │   ├── posts.py
│   │   │   │   └── auth.py
│   │   │   └── dependencies.py
│   │   └── v2/                 # API version 2 (if needed)
│   │
│   ├── models/                 # SQLAlchemy models
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── post.py
│   │   └── base.py
│   │
│   ├── schemas/                # Pydantic schemas
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── post.py
│   │   └── common.py
│   │
│   ├── services/               # Business logic layer
│   │   ├── __init__.py
│   │   ├── user_service.py
│   │   ├── post_service.py
│   │   └── auth_service.py
│   │
│   ├── repositories/           # Data access layer (optional)
│   │   ├── __init__.py
│   │   ├── user_repository.py
│   │   └── post_repository.py
│   │
│   ├── core/                   # Core functionality
│   │   ├── __init__.py
│   │   ├── security.py         # Auth utils
│   │   ├── config.py           # Settings
│   │   └── database.py         # DB connection
│   │
│   └── utils/                  # Utility functions
│       ├── __init__.py
│       ├── email.py
│       └── helpers.py
│
├── alembic/                    # Database migrations
│   ├── versions/
│   └── env.py
│
├── tests/                      # Test files
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_users.py
│   └── test_posts.py
│
├── .env                        # Environment variables (gitignored)
├── .env.example                # Example environment variables
├── .gitignore
├── alembic.ini
├── pyproject.toml              # Dependencies and config
├── pytest.ini
└── README.md
```

## 2. Layered Architecture

### Router → Service → Repository Pattern

**Router Layer (API Endpoints):**
```python
# app/api/v1/endpoints/users.py
from fastapi import APIRouter, Depends, HTTPException, status
from app.services.user_service import UserService
from app.schemas.user import UserCreate, UserResponse
from app.api.v1.dependencies import get_user_service

router = APIRouter(prefix="/users", tags=["users"])

@router.post("/", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def create_user(
    user_data: UserCreate,
    user_service: UserService = Depends(get_user_service)
):
    """Route handler - minimal logic, delegates to service"""
    return await user_service.create_user(user_data)

@router.get("/{user_id}", response_model=UserResponse)
async def get_user(
    user_id: int,
    user_service: UserService = Depends(get_user_service)
):
    user = await user_service.get_user(user_id)
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

**Service Layer (Business Logic):**
```python
# app/services/user_service.py
from sqlalchemy.ext.asyncio import AsyncSession
from app.repositories.user_repository import UserRepository
from app.schemas.user import UserCreate, UserUpdate
from app.core.security import hash_password
from typing import Optional

class UserService:
    def __init__(self, db: AsyncSession):
        self.repository = UserRepository(db)

    async def create_user(self, user_data: UserCreate):
        """Business logic for user creation"""
        # Check if user exists
        existing_user = await self.repository.get_by_email(user_data.email)
        if existing_user:
            raise ValueError("Email already registered")

        # Hash password
        hashed_password = hash_password(user_data.password)

        # Create user
        user_dict = user_data.model_dump(exclude={'password'})
        user_dict['hashed_password'] = hashed_password

        return await self.repository.create(user_dict)

    async def get_user(self, user_id: int):
        return await self.repository.get_by_id(user_id)

    async def update_user(self, user_id: int, user_data: UserUpdate):
        user = await self.repository.get_by_id(user_id)
        if not user:
            return None
        return await self.repository.update(user, user_data.model_dump(exclude_unset=True))
```

**Repository Layer (Data Access):**
```python
# app/repositories/user_repository.py
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select
from app.models.user import User
from typing import Optional, Dict, Any

class UserRepository:
    def __init__(self, db: AsyncSession):
        self.db = db

    async def create(self, user_data: Dict[str, Any]) -> User:
        user = User(**user_data)
        self.db.add(user)
        await self.db.flush()
        await self.db.refresh(user)
        return user

    async def get_by_id(self, user_id: int) -> Optional[User]:
        return await self.db.get(User, user_id)

    async def get_by_email(self, email: str) -> Optional[User]:
        stmt = select(User).where(User.email == email)
        result = await self.db.execute(stmt)
        return result.scalar_one_or_none()

    async def update(self, user: User, user_data: Dict[str, Any]) -> User:
        for key, value in user_data.items():
            setattr(user, key, value)
        await self.db.flush()
        await self.db.refresh(user)
        return user

    async def delete(self, user: User) -> None:
        await self.db.delete(user)
        await self.db.flush()
```

## 3. Dependency Injection

### Service Dependencies

**Good Example:**
```python
# app/api/v1/dependencies.py
from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession
from app.core.database import get_db
from app.services.user_service import UserService
from app.services.post_service import PostService

def get_user_service(db: AsyncSession = Depends(get_db)) -> UserService:
    return UserService(db)

def get_post_service(db: AsyncSession = Depends(get_db)) -> PostService:
    return PostService(db)
```

## 4. Configuration Management

**Good Example:**
```python
# app/core/config.py
from pydantic_settings import BaseSettings
from functools import lru_cache
from typing import Optional

class Settings(BaseSettings):
    # Application
    APP_NAME: str = "My FastAPI App"
    APP_VERSION: str = "1.0.0"
    DEBUG: bool = False

    # Database
    DATABASE_URL: str

    # Security
    SECRET_KEY: str
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30

    # CORS
    ALLOWED_ORIGINS: list[str] = ["http://localhost:3000"]

    # Email
    SMTP_HOST: Optional[str] = None
    SMTP_PORT: Optional[int] = None

    class Config:
        env_file = ".env"
        case_sensitive = True

@lru_cache()
def get_settings() -> Settings:
    return Settings()
```

## 5. Middleware Organization

**Good Example:**
```python
# app/core/middleware.py
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware
import time
import logging

logger = logging.getLogger(__name__)

class TimingMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        start_time = time.time()
        response = await call_next(request)
        process_time = time.time() - start_time
        response.headers["X-Process-Time"] = str(process_time)
        return response

# app/main.py - Register middleware
from app.core.middleware import TimingMiddleware
app.add_middleware(TimingMiddleware)
```

## 6. Exception Handling

**Good Example:**
```python
# app/core/exceptions.py
from fastapi import HTTPException, Request
from fastapi.responses import JSONResponse

class AppException(Exception):
    def __init__(self, message: str, status_code: int = 400):
        self.message = message
        self.status_code = status_code

class UserNotFoundError(AppException):
    def __init__(self, user_id: int):
        super().__init__(f"User {user_id} not found", status_code=404)

# app/main.py - Register exception handlers
@app.exception_handler(AppException)
async def app_exception_handler(request: Request, exc: AppException):
    return JSONResponse(
        status_code=exc.status_code,
        content={"message": exc.message}
    )
```

## Review Checklist

- [ ] Clear separation of concerns (router/service/repository)
- [ ] Business logic in service layer, not in routes
- [ ] Dependency injection used for services
- [ ] Configuration managed through environment variables
- [ ] Consistent project structure
- [ ] Middleware properly organized
- [ ] Custom exceptions defined
- [ ] API versioning implemented
- [ ] Utils and helpers properly modularized

## Common Issues to Look For

1. **Fat Routes** - Business logic in route handlers
2. **No Layering** - Database queries directly in routes
3. **Global State** - Services created as global variables
4. **Hardcoded Config** - Configuration not in environment variables
5. **Inconsistent Structure** - Files scattered without clear organization
