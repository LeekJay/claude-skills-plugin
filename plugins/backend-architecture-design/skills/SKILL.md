---
name: backend-architecture-design
description: Review and redesign Python FastAPI backend project architecture, analyzing API design, data architecture, security, async performance, code organization, and testing. Use when backend architecture review, API design analysis, database optimization, security assessment, or architectural improvements are needed. Also use when the user mentions backend architecture is unreasonable, needs restructuring, wants to refactor, or improve API/database design.
allowed-tools: Read, Grep, Glob
---

# Backend Architecture Design & Review

Comprehensive Python FastAPI backend project architecture review from six key dimensions: API design, data architecture, security, async performance, code organization, and testing & quality.

## Core Review Dimensions

### 1. API Design
- RESTful API design patterns and naming conventions
- Pydantic model design (Request/Response/Internal models)
- Dependency injection with `Depends()`
- OpenAPI documentation completeness
- API versioning and error response standardization

See [API_DESIGN.md](API_DESIGN.md) for details

### 2. Data Architecture
- SQLAlchemy/Tortoise ORM model design
- Database query optimization and N+1 problem detection
- Alembic migration management
- Database connection pooling configuration
- Async ORM usage patterns

See [DATA_ARCHITECTURE.md](DATA_ARCHITECTURE.md) for details

### 3. Security
- OAuth2/JWT authentication implementation
- Authorization and permission models
- Pydantic validation patterns
- CORS configuration and injection prevention
- Password hashing and sensitive data handling

See [SECURITY.md](SECURITY.md) for details

### 4. Async Performance
- Proper async/await usage (`async def` vs `def`)
- BackgroundTasks for non-blocking operations
- Database connection pooling
- Caching strategies and response optimization
- Avoiding blocking operations in async context

See [ASYNC_PERFORMANCE.md](ASYNC_PERFORMANCE.md) for details

### 5. Code Organization
- Layered architecture (router/service/repository pattern)
- Dependency injection patterns
- Middleware usage and exception handling
- Configuration management
- Business logic separation from routes

See [CODE_ORGANIZATION.md](CODE_ORGANIZATION.md) for details

### 6. Testing & Quality
- pytest test coverage and API testing
- Mock and fixture usage
- Type annotation completeness (mypy)
- Code formatting (black, ruff)
- Linting standards

See [TESTING.md](TESTING.md) for details

## Architecture Review Process

### Step 1: Project Overview
1. Use Glob to find project configuration files:
   - `pyproject.toml` / `requirements.txt` - Dependencies
   - `alembic.ini` - Database migrations
   - `.env.example` / `config.py` - Configuration
   - `docker-compose.yml` - Containerization
   - `pytest.ini` - Testing configuration

2. Use Read to examine configuration files and understand:
   - Tech stack (FastAPI version, ORM, database)
   - Main dependencies and versions
   - Database migration strategy
   - Configuration management approach

3. Use Glob to review directory structure:
   - `app/**/*` or `src/**/*` - Source code organization
   - `alembic/versions/*` - Migration files
   - `tests/**/*` - Test coverage

### Step 2: Dimension-Specific Deep Analysis

Choose relevant dimensions based on project needs for in-depth analysis:

**API Design Analysis**:
- Check router organization and endpoint naming
- Analyze Pydantic model design
- Find dependency injection usage
- Review response models and status codes

**Data Architecture Analysis**:
- Check ORM model relationships and indexes
- Find N+1 query problems
- Review Alembic migration files
- Analyze database session management

**Security Analysis**:
- Find authentication/authorization implementation
- Check CORS middleware configuration
- Search for SQL injection vulnerabilities
- Review password hashing implementation

**Async Performance Analysis**:
- Find blocking sync operations in async routes
- Check BackgroundTasks usage
- Review database connection pool settings
- Analyze caching implementation

**Code Organization Analysis**:
- Evaluate directory structure and layering
- Check separation of concerns
- Review dependency injection patterns
- Analyze exception handling approach

**Testing & Quality Analysis**:
- Review test coverage
- Check API test completeness
- Find missing type annotations
- Verify code formatting configuration

### Step 3: Generate Review Report

Organize findings and improvement suggestions in the following structure:

```markdown
# Backend Architecture Review Report

## Project Overview
- Framework: FastAPI [version]
- Python Version: [version]
- ORM: [SQLAlchemy/Tortoise/Other]
- Database: [PostgreSQL/MySQL/SQLite]
- Main Dependencies: [list key dependencies]

## Architecture Scores

| Dimension | Score | Notes |
|-----------|-------|-------|
| API Design | ⭐⭐⭐⭐☆ | Brief explanation |
| Data Architecture | ⭐⭐⭐☆☆ | Brief explanation |
| Security | ⭐⭐⭐⭐⭐ | Brief explanation |
| Async Performance | ⭐⭐⭐⭐☆ | Brief explanation |
| Code Organization | ⭐⭐⭐☆☆ | Brief explanation |
| Testing & Quality | ⭐⭐⭐⭐☆ | Brief explanation |

## Issues Found

### API Design Related
1. **Issue Description**
   - Location: `app/routers/users.py:45`
   - Impact: Medium
   - Recommendation: Specific improvement approach

### Data Architecture Related
[...]

### Security Related
[...]

### Async Performance Related
[...]

### Code Organization Related
[...]

### Testing & Quality Related
[...]

## Improvement Priorities

### P0 (Must fix immediately)
- [ ] Issue description and location

### P1 (Should fix short-term)
- [ ] Issue description and location

### P2 (Medium to long-term optimization)
- [ ] Issue description and location

## Architecture Recommendations

### Short-term Improvements
1. Specific recommendations with implementation steps

### Medium-term Optimizations
1. Specific recommendations with implementation steps

### Long-term Planning
1. Specific recommendations with implementation steps
```

## Review Best Practices

### Systematic Analysis
- Don't just look at surface issues, analyze root causes
- Focus on architectural decision trade-offs
- Consider team size and project stage
- Provide recommendations aligned with business needs

### Objective Evaluation
- Use specific code locations and examples
- Provide quantifiable metrics (response time, query count)
- Distinguish issue severity levels
- Offer actionable improvement plans

### Progressive Improvement
- Don't propose too many improvements at once
- Prioritize issues with highest impact
- Consider risk and cost of changes
- Provide phased implementation plans

### FastAPI-Specific Principles
- Leverage FastAPI's built-in features (automatic docs, validation)
- Focus on async best practices
- Emphasize type safety with Pydantic
- Consider performance implications of sync vs async

## Common Architecture Anti-patterns

### Anti-pattern 1: Fat Router
**Symptoms**: Route handlers contain extensive business logic

**Identification**:
- Use Read to check router files, code lines > 200
- Route functions contain database queries
- Complex business logic inside endpoint handlers

**Recommendations**:
- Extract business logic to service layer
- Use dependency injection for services
- Implement repository pattern for data access

### Anti-pattern 2: Sync in Async Context
**Symptoms**: Blocking synchronous operations in async routes

**Identification**:
- Use Grep to find `def` functions called in `async def` routes
- Look for synchronous I/O operations
- Check for CPU-intensive operations in async context

**Recommendations**:
- Use `async def` for I/O-bound operations
- Use `run_in_executor` for CPU-bound tasks
- Implement proper async database drivers

### Anti-pattern 3: Missing Dependency Injection
**Symptoms**: Hard-coded dependencies and global state

**Identification**:
- Use Grep to search for global variables
- Check if database sessions are passed as arguments
- Look for hard-coded configuration values

**Recommendations**:
- Use FastAPI's `Depends()` for dependency injection
- Create dependency factories for database sessions
- Implement configuration dependency providers

### Anti-pattern 4: Poor Pydantic Model Design
**Symptoms**: Reusing same model for request/response/database

**Identification**:
- Use Grep to find single Pydantic models used everywhere
- Check for models with `orm_mode` used as request models
- Look for models with optional fields that should be required

**Recommendations**:
- Separate Request, Response, and Internal models
- Use Pydantic inheritance for model variants
- Apply proper validators and field constraints

### Anti-pattern 5: N+1 Query Problem
**Symptoms**: Inefficient database queries causing performance issues

**Identification**:
- Use Grep to find loops with database queries
- Check for missing `joinedload()` or `selectinload()`
- Look for queries inside list comprehensions

**Recommendations**:
- Use eager loading with `joinedload()`
- Implement batch queries
- Add query monitoring and logging

### Anti-pattern 6: No Database Session Management
**Symptoms**: Improper database session lifecycle handling

**Identification**:
- Check if sessions are manually created/closed
- Look for sessions created as global variables
- Find missing session cleanup in exception handlers

**Recommendations**:
- Use dependency injection for database sessions
- Implement proper session cleanup with `finally`
- Use context managers for session management

### Anti-pattern 7: Missing Error Handling
**Symptoms**: Generic 500 errors without proper exception handling

**Identification**:
- Use Grep to find routes without try/except
- Check for missing custom exception handlers
- Look for unhandled validation errors

**Recommendations**:
- Implement custom exception handlers
- Use FastAPI's `HTTPException` properly
- Create domain-specific exception classes

### Anti-pattern 8: Weak Security Practices
**Symptoms**: Missing or improper authentication/authorization

**Identification**:
- Use Grep to find routes without authentication
- Check for passwords stored in plain text
- Look for missing CORS configuration

**Recommendations**:
- Implement OAuth2/JWT authentication
- Use password hashing (bcrypt/passlib)
- Configure CORS middleware properly
- Add rate limiting

## Usage Examples

### Example 1: Comprehensive Architecture Review
```
Please review the FastAPI backend architecture of the current project, focusing on API design and security.
```

### Example 2: Targeted Analysis
```
Analyze the project's database query patterns and identify N+1 problems.
```

### Example 3: Performance Optimization
```
Review the async/await usage in the project and find blocking operations.
```

### Example 4: Security Assessment
```
Conduct a security review of the authentication and authorization implementation.
```

## Output Standards

Each review should include:

1. **Specific Code Locations** - Use `file_path:line_number` format
2. **Impact Assessment** - High/Medium/Low
3. **Improvement Recommendations** - Actionable specific approaches
4. **Priority Ranking** - P0/P1/P2
5. **Reference Documentation** - Links to detailed best practice documents

## Important Notes

- This Skill is for analysis and recommendations only; it will not modify any code
- Review results are for reference; final decisions should consider team's actual situation
- Provides FastAPI-specific best practices tailored to Python ecosystem
- Focus on architectural evolution and security
- Consider team's technical capabilities and learning costs
