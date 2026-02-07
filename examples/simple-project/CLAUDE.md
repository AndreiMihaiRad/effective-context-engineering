# CLAUDE.md

FastAPI REST API for user management.

## Project Overview

**Architecture**: Hexagonal (domain, adapters, ports)  
**Tech Stack**: Python 3.12, FastAPI, SQLAlchemy, PostgreSQL, pytest

## Development Commands

```bash
# Setup
poetry install
poetry shell

# Development
poetry run uvicorn main:app --reload  # Dev server on http://localhost:8000
poetry run uvicorn main:app --reload --port 8080  # Custom port

# Testing
poetry run pytest                    # All tests
poetry run pytest tests/unit/        # Unit tests only
poetry run pytest tests/integration/ # Integration tests only
poetry run pytest -v                 # Verbose
poetry run pytest -k test_user       # Specific test

# Code Quality
poetry run black .                   # Format
poetry run isort .                   # Sort imports
poetry run mypy .                    # Type check
poetry run pylint src/               # Lint

# Database
alembic upgrade head                 # Run migrations
alembic revision --autogenerate -m "message"  # Create migration
```

## Architecture

### Hexagonal Design

```
src/
├── domain/           # Business logic (pure Python)
│   ├── models.py     # Domain models
│   └── services.py   # Business services
├── adapters/         # External integrations
│   ├── api/          # FastAPI routes
│   ├── db/           # Database repositories
│   └── external/     # External APIs
└── ports/            # Interface definitions
    ├── repositories.py
    └── services.py
```

**Key principle**: Domain has no dependencies on adapters.

### Request Flow

```
Request → API Route → Domain Service → Repository → Database
        ← Response ←                 ←            ←
```

## Memory-Bank Navigation

**Start with** `memory-bank/INDEX.md` for detailed documentation.

- Architecture: `memory-bank/architecture.md` (120 lines)
- API Patterns: `memory-bank/api-patterns.md` (185 lines)
- Testing: `memory-bank/testing.md` (95 lines)
- Deployment: `memory-bank/deployment.md` (110 lines)

## Code Standards

- **Formatting**: Black + isort
- **Type Hints**: Required for all functions
- **Docstrings**: Google style
- **Imports**: Absolute, sorted
- **Line Length**: 100 characters

## Quick Reference

### Adding New Endpoint

See `memory-bank/api-patterns.md` for full guide.

Quick version:
1. Create Pydantic models (request/response)
2. Add route in `src/adapters/api/`
3. Implement domain service in `src/domain/services.py`
4. Add repository method if needed
5. Write tests (unit + integration)

### Running Tests

```bash
# Fast: Unit tests only
poetry run pytest tests/unit/ -v

# Comprehensive: All tests with coverage
poetry run pytest --cov=src tests/

# Watch mode (requires pytest-watch)
poetry run ptw tests/
```

## Critical Constraints

### Security
- ❌ Never store passwords in plain text
- ❌ Never trust user input without validation
- ✅ Use Pydantic for all input validation
- ✅ Use password hashing (bcrypt)
- ✅ Validate JWT tokens in middleware

### Performance
- ❌ No sync database calls in async endpoints
- ❌ No N+1 queries (use eager loading)
- ✅ Use async for all I/O operations
- ✅ Add database indexes for query columns
- ✅ Use connection pooling

### Breaking Changes
- ❌ Don't change existing endpoint responses
- ❌ Don't remove optional parameters
- ✅ Add new endpoints for new behavior
- ✅ Version APIs if breaking changes needed
