# Data Architecture Best Practices for FastAPI

This guide covers database design, ORM usage, query optimization, migrations, and connection management for FastAPI applications.

## 1. ORM Model Design

### SQLAlchemy Model Best Practices

**Good Example:**
```python
from sqlalchemy import Column, Integer, String, Boolean, DateTime, ForeignKey, Index
from sqlalchemy.orm import relationship, Mapped, mapped_column
from sqlalchemy.sql import func
from datetime import datetime
from typing import List

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    email: Mapped[str] = mapped_column(String(255), unique=True, index=True, nullable=False)
    username: Mapped[str] = mapped_column(String(50), unique=True, index=True, nullable=False)
    hashed_password: Mapped[str] = mapped_column(String(255), nullable=False)
    is_active: Mapped[bool] = mapped_column(Boolean, default=True, nullable=False)
    is_superuser: Mapped[bool] = mapped_column(Boolean, default=False, nullable=False)

    # Timestamps with automatic updates
    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False
    )
    updated_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        onupdate=func.now(),
        nullable=False
    )

    # Relationships
    posts: Mapped[List["Post"]] = relationship(
        "Post",
        back_populates="author",
        cascade="all, delete-orphan",
        lazy="selectin"  # or "joined" depending on use case
    )

    # Composite indexes
    __table_args__ = (
        Index('idx_user_email_active', 'email', 'is_active'),
    )

    def __repr__(self):
        return f"<User(id={self.id}, username={self.username})>"


class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)
    title: Mapped[str] = mapped_column(String(200), nullable=False, index=True)
    content: Mapped[str] = mapped_column(Text, nullable=False)
    published: Mapped[bool] = mapped_column(Boolean, default=False, nullable=False)

    # Foreign key with proper indexing
    author_id: Mapped[int] = mapped_column(
        ForeignKey("users.id", ondelete="CASCADE"),
        nullable=False,
        index=True
    )

    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False
    )
    updated_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        onupdate=func.now(),
        nullable=False
    )

    # Relationships
    author: Mapped["User"] = relationship("User", back_populates="posts")
    comments: Mapped[List["Comment"]] = relationship(
        "Comment",
        back_populates="post",
        cascade="all, delete-orphan"
    )

    __table_args__ = (
        Index('idx_post_author_published', 'author_id', 'published'),
    )
```

**Anti-pattern: Poor Model Design**
```python
# ❌ Bad: Missing indexes, no timestamps, unclear relationships
class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)  # No index=True
    email = Column(String)  # No length limit, no unique constraint
    password = Column(String)  # Should be hashed_password
    # Missing created_at, updated_at
    # Missing is_active, is_deleted flags

    # No cascade rules
    posts = relationship("Post")
```

## 2. Query Optimization

### Avoiding N+1 Query Problem

**Problem: N+1 Queries**
```python
# ❌ Bad: This creates N+1 queries
@app.get("/users-with-posts")
async def get_users_with_posts(db: AsyncSession = Depends(get_db)):
    # 1 query to get all users
    result = await db.execute(select(User))
    users = result.scalars().all()

    response = []
    for user in users:  # N queries (one per user)
        response.append({
            "user": user,
            "posts": user.posts  # This triggers a separate query for each user!
        })
    return response
```

**Solution 1: Eager Loading with selectinload**
```python
from sqlalchemy.orm import selectinload

# ✅ Good: Uses 2 queries total (1 for users, 1 for all posts)
@app.get("/users-with-posts")
async def get_users_with_posts(db: AsyncSession = Depends(get_db)):
    stmt = select(User).options(selectinload(User.posts))
    result = await db.execute(stmt)
    users = result.scalars().all()
    return users
```

**Solution 2: Eager Loading with joinedload**
```python
from sqlalchemy.orm import joinedload

# ✅ Good: Uses 1 query with JOIN
@app.get("/users-with-posts")
async def get_users_with_posts(db: AsyncSession = Depends(get_db)):
    stmt = select(User).options(joinedload(User.posts))
    result = await db.execute(stmt)
    users = result.unique().scalars().all()  # unique() is important with joinedload
    return users
```

**Solution 3: Explicit JOIN Query**
```python
from sqlalchemy import select

# ✅ Good: Manual join with custom query
@app.get("/posts-with-authors")
async def get_posts_with_authors(db: AsyncSession = Depends(get_db)):
    stmt = (
        select(Post, User)
        .join(User, Post.author_id == User.id)
        .where(Post.published == True)
    )
    result = await db.execute(stmt)
    return [{"post": post, "author": author} for post, author in result]
```

### Pagination and Filtering

**Good Example:**
```python
from typing import Optional
from pydantic import BaseModel, Field

class PaginationParams(BaseModel):
    skip: int = Field(0, ge=0, description="Number of records to skip")
    limit: int = Field(100, ge=1, le=100, description="Max number of records to return")

class UserFilter(BaseModel):
    is_active: Optional[bool] = None
    search: Optional[str] = None

@app.get("/users")
async def list_users(
    pagination: PaginationParams = Depends(),
    filters: UserFilter = Depends(),
    db: AsyncSession = Depends(get_db)
):
    stmt = select(User)

    # Apply filters
    if filters.is_active is not None:
        stmt = stmt.where(User.is_active == filters.is_active)

    if filters.search:
        search_pattern = f"%{filters.search}%"
        stmt = stmt.where(
            (User.username.ilike(search_pattern)) |
            (User.email.ilike(search_pattern))
        )

    # Count total before pagination
    count_stmt = select(func.count()).select_from(stmt.subquery())
    total = await db.scalar(count_stmt)

    # Apply pagination
    stmt = stmt.offset(pagination.skip).limit(pagination.limit)

    result = await db.execute(stmt)
    users = result.scalars().all()

    return {
        "total": total,
        "items": users,
        "skip": pagination.skip,
        "limit": pagination.limit
    }
```

### Batch Operations

**Good Example:**
```python
from sqlalchemy.dialects.postgresql import insert

# Bulk insert with conflict handling
@app.post("/users/bulk")
async def bulk_create_users(
    users: List[UserCreate],
    db: AsyncSession = Depends(get_db)
):
    # Create list of user dicts
    users_data = [
        {
            "email": user.email,
            "username": user.username,
            "hashed_password": hash_password(user.password),
        }
        for user in users
    ]

    # Bulk insert (PostgreSQL specific)
    stmt = insert(User).values(users_data)
    stmt = stmt.on_conflict_do_nothing(index_elements=['email'])

    await db.execute(stmt)
    await db.commit()

    return {"created": len(users_data)}

# Bulk update
@app.patch("/posts/publish-all")
async def publish_all_posts(
    post_ids: List[int],
    db: AsyncSession = Depends(get_db)
):
    stmt = (
        update(Post)
        .where(Post.id.in_(post_ids))
        .values(published=True, updated_at=func.now())
    )
    result = await db.execute(stmt)
    await db.commit()

    return {"updated": result.rowcount}
```

## 3. Database Session Management

### Async Session Factory

**Good Example:**
```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession
from contextlib import asynccontextmanager

# Engine configuration
engine = create_async_engine(
    DATABASE_URL,
    echo=False,  # Set to True for debugging
    pool_size=20,  # Default connection pool size
    max_overflow=10,  # Extra connections when pool is exhausted
    pool_pre_ping=True,  # Verify connections before using
    pool_recycle=3600,  # Recycle connections after 1 hour
)

# Session factory
async_session_maker = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False,  # Don't expire objects after commit
    autocommit=False,
    autoflush=False,
)

# Dependency for route handlers
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
```

### Transaction Management

**Good Example:**
```python
from sqlalchemy.exc import IntegrityError

@app.post("/transfer")
async def transfer_funds(
    transfer: TransferRequest,
    db: AsyncSession = Depends(get_db)
):
    try:
        # Start explicit transaction
        async with db.begin():
            # Deduct from sender
            sender = await db.get(Account, transfer.sender_id)
            if sender.balance < transfer.amount:
                raise HTTPException(400, "Insufficient funds")
            sender.balance -= transfer.amount

            # Add to receiver
            receiver = await db.get(Account, transfer.receiver_id)
            receiver.balance += transfer.amount

            # Create transaction record
            transaction = Transaction(
                sender_id=transfer.sender_id,
                receiver_id=transfer.receiver_id,
                amount=transfer.amount
            )
            db.add(transaction)

            # Commit happens automatically when exiting the context

        return {"status": "success", "transaction_id": transaction.id}

    except IntegrityError as e:
        raise HTTPException(400, f"Database integrity error: {str(e)}")
```

## 4. Database Migrations with Alembic

### Migration File Best Practices

**Good Example:**
```python
"""Add user email verification

Revision ID: abc123def456
Revises: previous_revision
Create Date: 2024-01-01 12:00:00.000000
"""
from alembic import op
import sqlalchemy as sa

revision = 'abc123def456'
down_revision = 'previous_revision'
branch_labels = None
depends_on = None

def upgrade() -> None:
    # Add columns
    op.add_column('users',
        sa.Column('email_verified', sa.Boolean(), nullable=False, server_default='false')
    )
    op.add_column('users',
        sa.Column('verification_token', sa.String(255), nullable=True)
    )
    op.add_column('users',
        sa.Column('verification_token_expires', sa.DateTime(timezone=True), nullable=True)
    )

    # Add index
    op.create_index('idx_verification_token', 'users', ['verification_token'])

def downgrade() -> None:
    # Drop index
    op.drop_index('idx_verification_token', table_name='users')

    # Drop columns
    op.drop_column('users', 'verification_token_expires')
    op.drop_column('users', 'verification_token')
    op.drop_column('users', 'email_verified')
```

### Data Migration

**Good Example:**
```python
"""Migrate legacy user roles to new permission system

Revision ID: xyz789abc123
"""
from alembic import op
import sqlalchemy as sa
from sqlalchemy.sql import table, column

revision = 'xyz789abc123'
down_revision = 'previous_revision'

def upgrade() -> None:
    # Create new table
    op.create_table(
        'user_permissions',
        sa.Column('id', sa.Integer(), primary_key=True),
        sa.Column('user_id', sa.Integer(), sa.ForeignKey('users.id'), nullable=False),
        sa.Column('permission', sa.String(50), nullable=False),
        sa.Column('created_at', sa.DateTime(timezone=True), server_default=sa.func.now()),
    )

    # Migrate data from old role column
    users_table = table('users',
        column('id', sa.Integer),
        column('role', sa.String),
    )
    permissions_table = table('user_permissions',
        column('user_id', sa.Integer),
        column('permission', sa.String),
    )

    # Get connection
    conn = op.get_bind()

    # Migrate admin users
    admin_users = conn.execute(
        sa.select(users_table.c.id).where(users_table.c.role == 'admin')
    ).fetchall()

    if admin_users:
        conn.execute(
            permissions_table.insert(),
            [{'user_id': user.id, 'permission': 'admin'} for user in admin_users]
        )

    # Drop old column
    op.drop_column('users', 'role')

def downgrade() -> None:
    # Add back old column
    op.add_column('users', sa.Column('role', sa.String(20), nullable=True))

    # Migrate data back (simplified)
    # ... reverse migration logic ...

    # Drop new table
    op.drop_table('user_permissions')
```

## 5. Indexes and Query Performance

### Index Strategy

**When to Add Indexes:**
```python
class User(Base):
    __tablename__ = "users"

    # ✅ Primary key (automatic index)
    id: Mapped[int] = mapped_column(primary_key=True)

    # ✅ Unique columns (add unique index)
    email: Mapped[str] = mapped_column(String(255), unique=True, index=True)
    username: Mapped[str] = mapped_column(String(50), unique=True, index=True)

    # ✅ Foreign keys (always index)
    organization_id: Mapped[int] = mapped_column(ForeignKey("organizations.id"), index=True)

    # ✅ Frequently filtered columns
    is_active: Mapped[bool] = mapped_column(Boolean, default=True, index=True)
    created_at: Mapped[datetime] = mapped_column(DateTime(timezone=True), index=True)

    # ✅ Composite indexes for common query patterns
    __table_args__ = (
        Index('idx_org_active', 'organization_id', 'is_active'),
        Index('idx_email_active', 'email', 'is_active'),
    )
```

### Query Analysis

**Using EXPLAIN to Analyze Queries:**
```python
from sqlalchemy import text

@app.get("/debug/query-plan")
async def analyze_query(db: AsyncSession = Depends(get_db)):
    # Build your query
    stmt = (
        select(User)
        .join(Post)
        .where(User.is_active == True)
        .where(Post.published == True)
    )

    # Get query string
    query_str = str(stmt.compile(compile_kwargs={"literal_binds": True}))

    # Run EXPLAIN
    explain_stmt = text(f"EXPLAIN ANALYZE {query_str}")
    result = await db.execute(explain_stmt)

    return {"query_plan": [row for row in result]}
```

## 6. Connection Pooling

### Pool Configuration

**Good Example:**
```python
from sqlalchemy.pool import NullPool, QueuePool

# For async web servers (recommended)
engine = create_async_engine(
    DATABASE_URL,
    poolclass=QueuePool,
    pool_size=20,  # Number of connections to maintain
    max_overflow=10,  # Additional connections when pool is full
    pool_timeout=30,  # Seconds to wait for a connection
    pool_recycle=3600,  # Recycle connections after 1 hour
    pool_pre_ping=True,  # Test connections before use
)

# For serverless/lambda (use NullPool)
engine_serverless = create_async_engine(
    DATABASE_URL,
    poolclass=NullPool,  # No connection pooling
)
```

## Review Checklist

When reviewing data architecture, check for:

- [ ] All tables have proper indexes (primary keys, foreign keys, frequently queried columns)
- [ ] Timestamps (created_at, updated_at) on all tables
- [ ] Proper use of eager loading (selectinload/joinedload) to avoid N+1 queries
- [ ] Pagination implemented for list endpoints
- [ ] Database sessions managed through dependency injection
- [ ] Proper transaction handling for multi-step operations
- [ ] Alembic migrations for all schema changes
- [ ] Connection pool properly configured
- [ ] Bulk operations for batch inserts/updates
- [ ] Proper cascade rules on relationships
- [ ] Indexes on foreign keys
- [ ] Composite indexes for common query patterns

## Common Issues to Look For

1. **N+1 Queries** - Use Grep to find `for` loops with `.all()` followed by relationship access
2. **Missing Indexes** - Foreign keys without `index=True`
3. **No Pagination** - List endpoints without `limit` and `offset`
4. **Global Sessions** - Database sessions created globally instead of using `Depends(get_db)`
5. **Missing Timestamps** - Tables without `created_at`/`updated_at`
6. **Poor Transaction Handling** - Multi-step operations without transaction management
7. **Synchronous ORM with Async Routes** - Using sync SQLAlchemy in `async def` routes
8. **No Migration Strategy** - Schema changes made directly without Alembic migrations
