# Testing Best Practices for FastAPI

This guide covers pytest setup, API testing, fixtures, mocking, test coverage, and CI/CD integration for FastAPI applications.

## 1. Pytest Setup

### Test Configuration

**Good Example:**
```python
# pytest.ini
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts =
    --strict-markers
    --cov=app
    --cov-report=html
    --cov-report=term-missing
    -v

# pyproject.toml
[tool.pytest.ini_options]
asyncio_mode = "auto"

[tool.coverage.run]
omit = [
    "*/tests/*",
    "*/migrations/*",
    "*/__init__.py",
]
```

## 2. Test Fixtures

### Database Fixtures

**Good Example:**
```python
# tests/conftest.py
import pytest
import pytest_asyncio
from httpx import AsyncClient
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker
from app.main import app
from app.core.database import get_db, Base

TEST_DATABASE_URL = "sqlite+aiosqlite:///:memory:"

@pytest_asyncio.fixture
async def test_engine():
    engine = create_async_engine(TEST_DATABASE_URL, echo=False)
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.create_all)
    yield engine
    async with engine.begin() as conn:
        await conn.run_sync(Base.metadata.drop_all)
    await engine.dispose()

@pytest_asyncio.fixture
async def test_db(test_engine):
    async_session = async_sessionmaker(
        test_engine, class_=AsyncSession, expire_on_commit=False
    )
    async with async_session() as session:
        yield session

@pytest_asyncio.fixture
async def client(test_db):
    async def override_get_db():
        yield test_db

    app.dependency_overrides[get_db] = override_get_db
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac
    app.dependency_overrides.clear()
```

## 3. API Testing

### Testing CRUD Operations

**Good Example:**
```python
# tests/test_users.py
import pytest
from httpx import AsyncClient

@pytest.mark.asyncio
async def test_create_user(client: AsyncClient):
    response = await client.post(
        "/api/v1/users",
        json={
            "email": "test@example.com",
            "username": "testuser",
            "password": "SecurePass123!",
        }
    )
    assert response.status_code == 201
    data = response.json()
    assert data["email"] == "test@example.com"
    assert "id" in data
    assert "hashed_password" not in data  # Security check

@pytest.mark.asyncio
async def test_get_user(client: AsyncClient, test_user):
    response = await client.get(f"/api/v1/users/{test_user.id}")
    assert response.status_code == 200
    data = response.json()
    assert data["id"] == test_user.id
    assert data["email"] == test_user.email

@pytest.mark.asyncio
async def test_get_nonexistent_user(client: AsyncClient):
    response = await client.get("/api/v1/users/99999")
    assert response.status_code == 404

@pytest.mark.asyncio
async def test_update_user(client: AsyncClient, authenticated_client, test_user):
    response = await authenticated_client.put(
        f"/api/v1/users/{test_user.id}",
        json={"username": "newusername"}
    )
    assert response.status_code == 200
    data = response.json()
    assert data["username"] == "newusername"
```

### Testing Authentication

**Good Example:**
```python
@pytest.mark.asyncio
async def test_login_success(client: AsyncClient, test_user):
    response = await client.post(
        "/api/v1/auth/token",
        data={
            "username": test_user.username,
            "password": "password123"  # Test password
        }
    )
    assert response.status_code == 200
    data = response.json()
    assert "access_token" in data
    assert data["token_type"] == "bearer"

@pytest.mark.asyncio
async def test_login_invalid_credentials(client: AsyncClient):
    response = await client.post(
        "/api/v1/auth/token",
        data={"username": "nonexistent", "password": "wrong"}
    )
    assert response.status_code == 401

@pytest.mark.asyncio
async def test_protected_route_without_token(client: AsyncClient):
    response = await client.get("/api/v1/users/me")
    assert response.status_code == 401
```

## 4. Fixtures for Test Data

**Good Example:**
```python
# tests/conftest.py
from app.models.user import User
from app.core.security import hash_password

@pytest_asyncio.fixture
async def test_user(test_db):
    user = User(
        email="testuser@example.com",
        username="testuser",
        hashed_password=hash_password("password123"),
        is_active=True
    )
    test_db.add(user)
    await test_db.commit()
    await test_db.refresh(user)
    return user

@pytest_asyncio.fixture
async def admin_user(test_db):
    user = User(
        email="admin@example.com",
        username="admin",
        hashed_password=hash_password("adminpass123"),
        is_active=True,
        is_superuser=True
    )
    test_db.add(user)
    await test_db.commit()
    await test_db.refresh(user)
    return user

@pytest_asyncio.fixture
async def authenticated_client(client, test_user):
    # Login to get token
    response = await client.post(
        "/api/v1/auth/token",
        data={"username": test_user.username, "password": "password123"}
    )
    token = response.json()["access_token"]

    # Add auth header
    client.headers["Authorization"] = f"Bearer {token}"
    return client
```

## 5. Mocking External Services

**Good Example:**
```python
from unittest.mock import AsyncMock, patch

@pytest.mark.asyncio
@patch('app.services.email_service.send_email')
async def test_user_creation_sends_email(mock_send_email, client):
    mock_send_email.return_value = AsyncMock()

    response = await client.post(
        "/api/v1/users",
        json={
            "email": "newuser@example.com",
            "username": "newuser",
            "password": "SecurePass123!"
        }
    )

    assert response.status_code == 201
    mock_send_email.assert_called_once()
```

## 6. Test Coverage

### Running Coverage

```bash
# Run tests with coverage
pytest --cov=app --cov-report=html --cov-report=term-missing

# View coverage report
open htmlcov/index.html
```

### Coverage Goals

- Aim for 80%+ coverage
- 100% coverage on critical paths (auth, payments)
- Exclude test files and migrations from coverage

## 7. CI/CD Integration

### GitHub Actions Example

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: |
          pip install poetry
          poetry install

      - name: Run tests
        run: poetry run pytest --cov=app --cov-report=xml

      - name: Upload coverage
        uses: codecov/codecov-action@v3
```

## Review Checklist

- [ ] Pytest configured with async support
- [ ] Test database fixtures set up
- [ ] Test coverage > 80%
- [ ] Authentication testing implemented
- [ ] CRUD operations tested
- [ ] External services mocked
- [ ] CI/CD pipeline configured
- [ ] Type annotations verified (mypy)

## Common Testing Issues

1. **No Test Database** - Tests using production database
2. **Missing Fixtures** - Duplicated test data creation
3. **Not Testing Auth** - Skipping authentication tests
4. **No Mocking** - Tests calling real external APIs
5. **Low Coverage** - Critical code paths not tested
