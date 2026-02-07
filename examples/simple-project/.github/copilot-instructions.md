# GitHub Copilot Custom Instructions

> This file provides guidance to GitHub Copilot when working with this repository.

## Project Overview

FastAPI REST API for user management with hexagonal architecture.

| Component | Purpose | Language/Version |
|-----------|---------|------------------|
| `src/` | Core application code | Python 3.12 |
| `tests/` | Unit and integration tests | pytest |

## Memory-Bank Navigation

**Start with** `memory-bank/INDEX.md` for detailed documentation.

- Architecture: `memory-bank/architecture.md` (120 lines)
- API Patterns: `memory-bank/api-patterns.md` (185 lines)
- Testing: `memory-bank/testing.md` (95 lines)
- Deployment: `memory-bank/deployment.md` (110 lines)

## Development Workflow

### Common Commands

```bash
# Setup
poetry install
poetry shell

# Development
poetry run uvicorn main:app --reload  # Dev server on http://localhost:8000

# Testing
poetry run pytest                     # All tests
poetry run pytest tests/unit/         # Unit tests only
poetry run pytest tests/integration/  # Integration tests only

# Code Quality
poetry run black .                    # Format
poetry run isort .                    # Sort imports
poetry run mypy .                     # Type check

# Database
alembic upgrade head                  # Run migrations
```

## Architecture

### Hexagonal Design (Ports & Adapters)

```
src/
├── domain/           # Business logic (pure Python)
├── adapters/         # External integrations (API, DB)
└── ports/            # Interface definitions
```

**Key principle**: Domain has no dependencies on adapters.

**Request flow**: API Route → Domain Service → Repository → Database

## Code Standards

**Language conventions**:
- **Formatting**: Black + isort (line length: 100)
- **Type annotations**: Required for all functions
- **Docstrings**: Google style
- **Testing**: pytest with unit and integration tests
- **Package management**: poetry

**Naming conventions**:
- Variables/functions: `snake_case`
- Classes: `PascalCase`
- Constants: `UPPER_CASE`
- Files: `snake_case.py`
- Test files: `test_*.py`

## Critical Constraints

### Security (DO NOT)
- ❌ Store passwords in plain text
- ❌ Trust user input without validation
- ❌ Expose sensitive data in error messages
- ✅ Use Pydantic for all input validation
- ✅ Use password hashing (bcrypt)
- ✅ Validate JWT tokens in middleware

### Performance (DO NOT)
- ❌ Use sync database calls in async endpoints
- ❌ Create N+1 queries (use eager loading)
- ❌ Skip database indexes on query columns
- ✅ Use async for all I/O operations
- ✅ Add indexes for filter columns
- ✅ Use connection pooling

### Breaking Changes (DO NOT)
- ❌ Change existing endpoint response schemas
- ❌ Remove optional parameters without versioning
- ❌ Rename API parameters
- ✅ Add new endpoints for new behavior
- ✅ Version APIs if breaking changes needed
- ✅ Deprecate old endpoints gracefully

## Quick Reference

### Adding New Endpoint
See `memory-bank/api-patterns.md` for complete guide.

Quick steps:
1. Create Pydantic models (request/response)
2. Add route in `src/adapters/api/`
3. Implement domain service
4. Add repository method if needed
5. Write tests (unit + integration)

### Running Tests
```bash
# Fast: Unit tests only
poetry run pytest tests/unit/ -v

# Comprehensive: All tests with coverage
poetry run pytest --cov=src tests/
```

### Debugging Issues
See `memory-bank/troubleshooting.md` for common issues and solutions.

---

**Note**: This file is kept under 200 lines. For comprehensive documentation, reference `memory-bank/`.
