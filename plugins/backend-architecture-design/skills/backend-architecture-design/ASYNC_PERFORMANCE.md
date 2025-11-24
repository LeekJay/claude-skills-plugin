# Async Performance Best Practices for FastAPI

This guide covers async/await patterns, background tasks, caching, database optimization, and performance monitoring for FastAPI applications.

## 1. Async vs Sync Route Handlers

### When to Use Async

**Good Examples:**
```python
# ✅ Use async for I/O-bound operations
@app.get("/users/{user_id}")
async def get_user(user_id: int, db: AsyncSession = Depends(get_db)):
    """Database query - I/O bound, use async"""
    user = await db.get(User, user_id)
    return user

@app.get("/external-api")
async def call_external_api():
    """HTTP requests - I/O bound, use async"""
    async with httpx.AsyncClient() as client:
        response = await client.get("https://api.example.com/data")
        return response.json()

@app.post("/send-email")
async def send_email(email_data: EmailData):
    """Email sending - I/O bound, use async"""
    await send_email_async(email_data)
    return {"status": "sent"}
```

**When to Use Sync:**
```python
# ✅ Use sync (def) for CPU-bound operations or when no async operations
@app.get("/calculate")
def heavy_calculation(n: int):
    """Pure CPU computation - no I/O, use sync"""
    result = complex_mathematical_operation(n)
    return {"result": result}

@app.get("/simple-response")
def simple_endpoint():
    """No I/O operations - can use sync"""
    return {"message": "Hello World"}
```

### Mixing Sync and Async (Anti-patterns)

**Problem: Blocking Operations in Async Routes**
```python
import time
import requests  # Synchronous library

# ❌ Bad: Blocking sync operation in async function
@app.get("/bad-example")
async def bad_async_route():
    # This blocks the event loop!
    time.sleep(5)  # DON'T DO THIS
    response = requests.get("https://api.example.com")  # DON'T DO THIS
    return response.json()
```

**Solution: Use Async Libraries or run_in_executor**
```python
import asyncio
import httpx
from concurrent.futures import ThreadPoolExecutor

# ✅ Good: Use async libraries
@app.get("/good-example")
async def good_async_route():
    await asyncio.sleep(5)  # Non-blocking sleep
    async with httpx.AsyncClient() as client:
        response = await client.get("https://api.example.com")
        return response.json()

# ✅ Good: Use run_in_executor for CPU-bound tasks
executor = ThreadPoolExecutor(max_workers=4)

def cpu_intensive_task(data):
    """Simulate CPU-intensive work"""
    import hashlib
    result = hashlib.pbkdf2_hmac('sha256', data.encode(), b'salt', 100000)
    return result.hex()

@app.post("/process")
async def process_data(data: str):
    loop = asyncio.get_event_loop()
    result = await loop.run_in_executor(executor, cpu_intensive_task, data)
    return {"result": result}
```

## 2. Background Tasks

### Using FastAPI BackgroundTasks

**Good Example:**
```python
from fastapi import BackgroundTasks
import logging

logger = logging.getLogger(__name__)

# Background function
async def send_welcome_email(email: str, username: str):
    """Send email in background"""
    await asyncio.sleep(2)  # Simulate email sending
    logger.info(f"Welcome email sent to {email}")

async def log_user_activity(user_id: int, action: str):
    """Log activity in background"""
    async with async_session_maker() as db:
        log_entry = ActivityLog(user_id=user_id, action=action)
        db.add(log_entry)
        await db.commit()

# Route with background tasks
@app.post("/users", status_code=201)
async def create_user(
    user: UserCreate,
    background_tasks: BackgroundTasks,
    db: AsyncSession = Depends(get_db)
):
    # Create user (blocking operation)
    db_user = User(**user.model_dump())
    db.add(db_user)
    await db.commit()
    await db.refresh(db_user)

    # Add background tasks
    background_tasks.add_task(send_welcome_email, db_user.email, db_user.username)
    background_tasks.add_task(log_user_activity, db_user.id, "user_created")

    return db_user
```

### When Background Tasks Are NOT Enough

**Use Celery for:**
```python
# For long-running tasks, use Celery/RQ/Dramatiq
from celery import Celery

celery_app = Celery('tasks', broker='redis://localhost:6379')

@celery_app.task
def process_large_file(file_path: str):
    """Long-running task that should be in queue"""
    # Process large file
    pass

@celery_app.task
def generate_report(user_id: int):
    """Report generation that takes minutes"""
    # Generate complex report
    pass

@app.post("/upload")
async def upload_file(file: UploadFile):
    # Save file
    file_path = save_file(file)

    # Queue task (don't wait for result)
    process_large_file.delay(file_path)

    return {"status": "processing", "message": "File uploaded and queued"}
```

## 3. Caching Strategies

### In-Memory Caching

**Good Example:**
```python
from functools import lru_cache
from fastapi_cache import FastAPICache
from fastapi_cache.backends.redis import RedisBackend
from fastapi_cache.decorator import cache
from redis import asyncio as aioredis

# Initialize Redis cache
@app.on_event("startup")
async def startup():
    redis = aioredis.from_url("redis://localhost")
    FastAPICache.init(RedisBackend(redis), prefix="fastapi-cache")

# Cache expensive database queries
@app.get("/users/{user_id}")
@cache(expire=300)  # Cache for 5 minutes
async def get_user(user_id: int, db: AsyncSession = Depends(get_db)):
    user = await db.get(User, user_id)
    if not user:
        raise HTTPException(404, "User not found")
    return user

# Cache computed results
@app.get("/statistics")
@cache(expire=3600)  # Cache for 1 hour
async def get_statistics(db: AsyncSession = Depends(get_db)):
    # Expensive aggregation query
    stats = await compute_statistics(db)
    return stats

# Python functools caching for config/settings
@lru_cache(maxsize=1)
def get_app_config():
    """Cache configuration (computed once)"""
    return load_config_from_file()
```

### Cache Invalidation

**Good Example:**
```python
from fastapi_cache import FastAPICache

@app.put("/users/{user_id}")
async def update_user(
    user_id: int,
    user_update: UserUpdate,
    db: AsyncSession = Depends(get_db)
):
    # Update user
    user = await db.get(User, user_id)
    for key, value in user_update.model_dump(exclude_unset=True).items():
        setattr(user, key, value)
    await db.commit()

    # Invalidate cache
    await FastAPICache.clear(namespace=f"get_user:{user_id}")

    return user
```

## 4. Database Connection Pooling

### Optimal Pool Configuration

**Good Example:**
```python
from sqlalchemy.ext.asyncio import create_async_engine

# Production configuration
engine = create_async_engine(
    DATABASE_URL,
    echo=False,
    pool_size=20,  # Number of permanent connections
    max_overflow=10,  # Additional connections when pool exhausted
    pool_timeout=30,  # Wait time for available connection (seconds)
    pool_recycle=3600,  # Recycle connections after 1 hour
    pool_pre_ping=True,  # Verify connection health before use
)

# For high-traffic applications
engine_high_traffic = create_async_engine(
    DATABASE_URL,
    pool_size=50,
    max_overflow=20,
    pool_timeout=10,
)

# Monitor pool status
@app.get("/health/db-pool")
async def db_pool_status():
    pool = engine.pool
    return {
        "size": pool.size(),
        "checked_in": pool.checkedin(),
        "checked_out": pool.checkedout(),
        "overflow": pool.overflow(),
    }
```

## 5. Query Optimization

### Efficient Queries

**Good Example:**
```python
from sqlalchemy import select, func
from sqlalchemy.orm import selectinload, joinedload

# ✅ Good: Efficient pagination with total count
@app.get("/posts")
async def list_posts(
    skip: int = 0,
    limit: int = 20,
    db: AsyncSession = Depends(get_db)
):
    # Get total count
    count_query = select(func.count()).select_from(Post)
    total = await db.scalar(count_query)

    # Get paginated results with eager loading
    query = (
        select(Post)
        .options(selectinload(Post.author))  # Avoid N+1
        .offset(skip)
        .limit(limit)
        .order_by(Post.created_at.desc())
    )
    result = await db.execute(query)
    posts = result.scalars().all()

    return {
        "total": total,
        "items": posts,
        "skip": skip,
        "limit": limit
    }

# ✅ Good: Use select_from for complex queries
@app.get("/active-users-with-posts")
async def active_users_with_posts(db: AsyncSession = Depends(get_db)):
    query = (
        select(User, func.count(Post.id).label('post_count'))
        .join(Post)
        .where(User.is_active == True)
        .group_by(User.id)
        .having(func.count(Post.id) > 0)
    )
    result = await db.execute(query)
    return [{"user": user, "post_count": count} for user, count in result]
```

## 6. HTTP Client Optimization

### Async HTTP Requests

**Good Example:**
```python
import httpx
from typing import List

# Reuse client (connection pooling)
http_client = httpx.AsyncClient(
    timeout=10.0,
    limits=httpx.Limits(max_keepalive_connections=20, max_connections=100)
)

@app.on_event("shutdown")
async def shutdown():
    await http_client.aclose()

# Single request
@app.get("/external-data/{id}")
async def get_external_data(id: int):
    response = await http_client.get(f"https://api.example.com/items/{id}")
    response.raise_for_status()
    return response.json()

# Parallel requests
@app.get("/multiple-external")
async def get_multiple_external(ids: List[int]):
    # Make requests concurrently
    tasks = [
        http_client.get(f"https://api.example.com/items/{id}")
        for id in ids
    ]
    responses = await asyncio.gather(*tasks, return_exceptions=True)

    results = []
    for response in responses:
        if isinstance(response, Exception):
            results.append({"error": str(response)})
        else:
            results.append(response.json())

    return results
```

## 7. Response Time Optimization

### Streaming Responses

**Good Example:**
```python
from fastapi.responses import StreamingResponse
import csv
from io import StringIO

@app.get("/export/users")
async def export_users(db: AsyncSession = Depends(get_db)):
    """Stream large CSV export"""
    async def generate():
        # Write header
        output = StringIO()
        writer = csv.writer(output)
        writer.writerow(['id', 'email', 'username'])
        yield output.getvalue()
        output.truncate(0)
        output.seek(0)

        # Stream users in batches
        offset = 0
        batch_size = 1000
        while True:
            query = select(User).offset(offset).limit(batch_size)
            result = await db.execute(query)
            users = result.scalars().all()

            if not users:
                break

            for user in users:
                writer.writerow([user.id, user.email, user.username])
                yield output.getvalue()
                output.truncate(0)
                output.seek(0)

            offset += batch_size

    return StreamingResponse(
        generate(),
        media_type="text/csv",
        headers={"Content-Disposition": "attachment; filename=users.csv"}
    )
```

### Response Compression

**Good Example:**
```python
from fastapi.middleware.gzip import GZipMiddleware

# Enable compression for responses > 1KB
app.add_middleware(GZipMiddleware, minimum_size=1000)
```

## 8. Performance Monitoring

### Request Timing Middleware

**Good Example:**
```python
import time
from fastapi import Request
import logging

logger = logging.getLogger(__name__)

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time

    response.headers["X-Process-Time"] = str(process_time)

    # Log slow requests
    if process_time > 1.0:  # 1 second threshold
        logger.warning(
            f"Slow request: {request.method} {request.url.path} "
            f"took {process_time:.2f}s"
        )

    return response
```

## Review Checklist

When reviewing async performance, check for:

- [ ] All I/O-bound operations use `async def`
- [ ] No blocking operations in async routes (time.sleep, requests.get, etc.)
- [ ] Background tasks used for non-critical operations
- [ ] Caching implemented for expensive queries
- [ ] Database connection pool properly configured
- [ ] N+1 queries avoided with eager loading
- [ ] HTTP client reused (not created per request)
- [ ] Parallel async operations use `asyncio.gather()`
- [ ] Streaming responses for large data exports
- [ ] Response compression enabled
- [ ] Performance monitoring in place
- [ ] Slow query logging enabled

## Common Performance Issues to Look For

1. **Blocking in Async** - Use Grep to find `time.sleep`, `requests.get` in async functions
2. **No Connection Pooling** - Database connections created per request
3. **Missing Caching** - Expensive queries without cache decorators
4. **N+1 Queries** - Loops accessing relationships without eager loading
5. **Sequential API Calls** - Multiple HTTP requests not parallelized
6. **Large Response Bodies** - No pagination or streaming for large datasets
7. **Missing Indexes** - Slow queries due to missing database indexes
8. **Synchronous Libraries** - Using `requests` instead of `httpx.AsyncClient`
