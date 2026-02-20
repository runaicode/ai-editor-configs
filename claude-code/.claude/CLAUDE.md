# Project Instructions for Claude Code

## Project Overview
This is a production SaaS application. You are a senior engineer working on this codebase.
Treat every change as if it will be deployed to production immediately.

## Technology Stack
- **Language**: Python 3.12
- **Framework**: FastAPI 0.115+
- **Database**: PostgreSQL 17 via SQLAlchemy 2.0 (async)
- **Cache**: Redis 7
- **Queue**: Celery with Redis broker
- **Frontend**: React 19 + TypeScript (separate repo)
- **Testing**: pytest, pytest-asyncio, httpx
- **CI/CD**: GitHub Actions
- **Infrastructure**: AWS (ECS, RDS, ElastiCache)

## Code Style & Conventions

### Python
- Follow PEP 8 strictly. Line length: 120 characters.
- Use type hints on ALL function signatures. No `Any` unless truly unavoidable.
- Use `async def` for all route handlers and database operations.
- Docstrings: Google style for all public functions.
- Imports: stdlib, then third-party, then local — separated by blank lines.
- Use `pathlib.Path` instead of `os.path`.
- Use f-strings, never `.format()` or `%` formatting.

### Naming
```
files:       snake_case.py (user_service.py, auth_middleware.py)
classes:     PascalCase (UserService, AuthMiddleware)
functions:   snake_case (get_user_by_id, validate_token)
constants:   UPPER_SNAKE (MAX_RETRIES, DEFAULT_PAGE_SIZE)
env vars:    UPPER_SNAKE (DATABASE_URL, REDIS_URL)
endpoints:   /api/v1/kebab-case (/api/v1/user-profiles)
db tables:   plural snake_case (users, user_profiles, audit_logs)
```

### Project Structure
```
app/
├── api/
│   ├── v1/
│   │   ├── routes/      # Route handlers (thin — call services)
│   │   ├── schemas/     # Pydantic request/response models
│   │   └── deps.py      # Dependency injection (get_db, get_current_user)
├── core/
│   ├── config.py        # Settings via pydantic-settings
│   ├── security.py      # JWT, password hashing
│   └── exceptions.py    # Custom exception classes
├── models/              # SQLAlchemy ORM models
├── services/            # Business logic (one service per domain)
├── repositories/        # Database queries (one repo per model)
├── tasks/               # Celery async tasks
├── middleware/           # Custom middleware
└── tests/
    ├── unit/            # Fast, isolated tests
    ├── integration/     # Tests with database
    └── conftest.py      # Fixtures
```

### Architecture Rules
1. **Routes are thin**: validate input, call service, return response. No business logic.
2. **Services contain logic**: one service per domain (UserService, BillingService).
3. **Repositories handle SQL**: services never write raw queries.
4. **Schemas validate I/O**: Pydantic models for all API input/output.
5. **Dependencies inject**: use FastAPI's `Depends()` for db sessions, auth, etc.

## Error Handling
- Custom exceptions in `core/exceptions.py` — never raise generic `Exception`.
- All API errors return: `{"detail": {"code": "ERROR_CODE", "message": "Human readable"}}`.
- Log errors with context: `logger.error("Failed to create user", extra={"email": email, "error": str(e)})`.
- Never expose stack traces or internal details to API clients.

## Security Rules
- NEVER hardcode secrets. Use `core/config.py` which reads from environment.
- NEVER log sensitive data (passwords, tokens, PII).
- NEVER use `eval()`, `exec()`, or `__import__()`.
- ALL user input must go through Pydantic validation.
- SQL: always use SQLAlchemy ORM or parameterized queries.
- Auth: JWT access tokens (15 min) + refresh tokens (7 days, stored in httpOnly cookie).
- Rate limit: all public endpoints via `slowapi`.

## Testing
- Every new function needs a test. Minimum 80% coverage.
- Use `pytest` fixtures for database sessions, authenticated clients, etc.
- Name tests descriptively: `test_create_user_returns_422_when_email_invalid`.
- Mock external services (email, payment, S3). Never call real APIs.
- Integration tests use a test database (auto-created/destroyed).

## Database
- Migrations: Alembic. Always create a migration for schema changes.
- Never modify existing migration files. Create new ones.
- Foreign keys: always set `ondelete` behavior explicitly.
- Indexes: add for any column used in WHERE, JOIN, or ORDER BY.
- Soft delete: use `deleted_at` timestamp, never hard delete user data.

## What NOT to Do
- Do NOT add dependencies without justification. Check if stdlib or existing deps solve it.
- Do NOT write synchronous database code. Everything is async.
- Do NOT put business logic in route handlers, models, or schemas.
- Do NOT return database models directly from API routes. Always use response schemas.
- Do NOT commit `.env` files, credentials, or generated files.
- Do NOT use `SELECT *` — always specify columns in repositories.
- Do NOT skip type hints to save time. They catch bugs and improve IDE support.

## Performance Guidelines
- Use `select_in_loading` or `joinedload` to avoid N+1 queries.
- Cache frequently-read, rarely-changed data in Redis (TTL: 5 minutes default).
- Use pagination for all list endpoints (default: 20, max: 100).
- Background tasks: use Celery for anything > 500ms (emails, reports, webhooks).

## Git Workflow
- Conventional commits: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`
- One logical change per commit.
- Run `pytest` and `ruff check` before every commit.
- PR title format: `[TICKET-123] feat: short description`
