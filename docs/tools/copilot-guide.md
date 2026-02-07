# GitHub Copilot: Context Engineering Guide

> **Key insight**: GitHub Copilot uses repository-specific custom instructions via `.github/copilot-instructions.md` combined with the memory-bank pattern for effective context management.

**Attribution**: Synthesized from context engineering patterns proven in production (nexus, data-api) and adapted for GitHub Copilot's architecture.

---

## Quick Start

### 1. Create Directory Structure

```bash
mkdir -p .github
```

### 2. Create Instructions File

Create `.github/copilot-instructions.md`:

```markdown
# GitHub Copilot Custom Instructions

## Project Overview
[Brief description]

## Memory-Bank Navigation
Start with `memory-bank/INDEX.md` for detailed documentation.

## Code Standards
- Formatting: [tool]
- Type annotations: [required/optional]
- Package management: [tool]

## Critical Constraints
[Key constraints that must never be violated]
```

### 3. Test It

Open any file in your repository with GitHub Copilot enabled. The instructions are automatically loaded for all contributors.

---

## Configuration Format

### File Location

**Single file**: `.github/copilot-instructions.md`

```
project/
├── .github/
│   └── copilot-instructions.md    # Repository-wide custom instructions
├── memory-bank/                   # Detailed documentation
│   ├── INDEX.md
│   ├── architecture/
│   ├── components/
│   └── workflows/
└── ...
```

### File Format

Plain Markdown (no frontmatter):

```markdown
# GitHub Copilot Custom Instructions

> This file provides guidance to GitHub Copilot when working with this repository.

## Project Overview
[2-3 sentence description]

## Memory-Bank Navigation
[Pointers to detailed docs]

## Code Standards
[Formatting, naming, testing]

## Critical Constraints
[Security, performance, breaking changes]
```

**Key characteristics**:
- Plain Markdown (simpler than Cursor's YAML frontmatter)
- Single file (vs. Cursor/Windsurf multiple rules)
- Always loaded (no conditional loading like Cursor/Windsurf globs)
- Repository-scoped (all contributors get same context)

---

## Core Principles

### 1. Keep It Under 200 Lines

**Why 200 lines?**
- GitHub Copilot loads the entire file for every session
- Shorter = faster context processing
- Forces focus on essentials
- Leaves context budget for code

**How to stay under 200**:
- Focus on constraints, not documentation
- Point to memory-bank for details
- Avoid duplicating information
- Remove generic advice

**Example**:

❌ **Too long** (400 lines):
```markdown
# Instructions (400 lines)

## Complete API Documentation (150 lines)
- Endpoint 1: Full details...
- Endpoint 2: Full details...
- Endpoint 3: Full details...

## Complete Architecture (100 lines)
[Exhaustive architecture description]

## All Transformation Logic (100 lines)
[Every transformation documented]
```

✅ **Concise** (150 lines):
```markdown
# Instructions (150 lines)

## Overview (20 lines)
[Brief project description]

## Critical Constraints (50 lines)
[What not to do + correct approaches]

## Memory-Bank Pointers (30 lines)
- API: `memory-bank/api-patterns.md`
- Architecture: `memory-bank/architecture/overview.md`

## Quick Reference (50 lines)
[Most common tasks with memory-bank links]
```

---

### 2. Focus on Constraints, Not Documentation

**Constraints** tell Copilot what NOT to do and what to do instead.
**Documentation** explains how things work (belongs in memory-bank).

**Bad** (documenting):
```markdown
## Database Access

Our application uses SQLAlchemy for database access. We have a User
model that represents users in the database. The model has fields for
id, name, email, and password_hash. To query users, we use the
session.query() method...
```

**Good** (constraining):
```markdown
## Database Constraints

- ❌ Never use sync queries in async endpoints
- ❌ Never expose password_hash in responses
- ❌ Avoid N+1 queries (use eager loading)
- ✅ Use async SQLAlchemy methods
- ✅ Add indexes on filter columns
- ✅ See `memory-bank/database-patterns.md` for details
```

---

### 3. Use Memory-Bank for Details

GitHub Copilot instructions should **point to** documentation, not **contain** it.

**Structure**:
```
.github/copilot-instructions.md   # Pointers + constraints (< 200 lines)
memory-bank/                       # Detailed docs (any size)
├── INDEX.md                       # Navigation hub
├── architecture/
├── components/
└── workflows/
```

**Example pointer section**:
```markdown
## Memory-Bank Navigation

**Start with** `memory-bank/INDEX.md` for task-based navigation.

### Quick Access by Task

| Task | Documentation | Size |
|------|---------------|------|
| Adding endpoint | `memory-bank/workflows/adding-endpoint.md` | 365 lines |
| Testing changes | `memory-bank/workflows/testing.md` | 360 lines |
| Debugging | `memory-bank/workflows/troubleshooting.md` | 468 lines |
| Understanding architecture | `memory-bank/architecture/overview.md` | 87 lines |

### By Component

- Trino Facade: `memory-bank/components/trino-facade.md` (263 lines)
- Kong Gateway: `memory-bank/components/kong-gateway.md` (221 lines)
```

---

### 4. Organize by Task, Not Structure

Help Copilot understand developer **intent**, not just project **structure**.

**Bad** (structural):
```markdown
## Project Structure

- src/domain/
- src/adapters/
- src/ports/
- tests/unit/
- tests/integration/
```

**Good** (task-based):
```markdown
## Common Tasks

### Adding New Endpoint
See `memory-bank/workflows/adding-endpoint.md`
1. Create Pydantic models
2. Add route
3. Implement service
4. Write tests

### Debugging Failed Requests
See `memory-bank/workflows/troubleshooting.md`
- Check logs: `kubectl logs -l app=service`
- Check traces: Datadog APM
- Review SQL: Check query syntax in template
```

---

## Key Differences from Other Tools

### GitHub Copilot vs. Cursor

| Aspect | GitHub Copilot | Cursor |
|--------|----------------|--------|
| **File location** | `.github/copilot-instructions.md` | `.cursor/rules/*.mdc` |
| **Format** | Plain Markdown | YAML + Markdown |
| **Multiple files** | No (single file) | Yes (multiple rules) |
| **Conditional loading** | No | Yes (via globs) |
| **Activation** | Always active | Per-file via globs or alwaysApply |
| **Recommended size** | < 200 lines | < 100 lines per file |

**When to use Cursor over Copilot**:
- Large projects needing file-specific rules
- Need conditional rule loading (only for .ts files, only for API/, etc.)
- Want multiple focused rule files instead of one

**When to use Copilot over Cursor**:
- Want native GitHub integration
- Simpler single-file setup
- Team uses various IDEs (VS Code, JetBrains, etc.)

---

### GitHub Copilot vs. Windsurf

| Aspect | GitHub Copilot | Windsurf |
|--------|----------------|----------|
| **File location** | `.github/copilot-instructions.md` | `.windsurf/rules/*.md` |
| **Format** | Plain Markdown | Plain Markdown |
| **Multiple files** | No | Yes |
| **Conditional loading** | No | Yes (via globs) |
| **Recommended size** | < 200 lines | < 75 lines per file |

**Key difference**: Windsurf loads all rules for every session (hence the 75-line recommendation), while GitHub Copilot also loads its single file always.

---

### GitHub Copilot vs. Claude Code

| Aspect | GitHub Copilot | Claude Code |
|--------|----------------|-------------|
| **File location** | `.github/copilot-instructions.md` | `CLAUDE.md` (hierarchy) |
| **Format** | Plain Markdown | Plain Markdown |
| **Hierarchy** | No | Yes (global → project → directory) |
| **Conditional loading** | No | Yes (directory-based) |
| **Recommended size** | < 200 lines | < 150 lines |
| **Scope** | Repository | Global + project + directory |

**Key difference**: Claude Code supports hierarchical context (global preferences → project → component), while GitHub Copilot is repository-only.

---

## Best Practices

### 1. Start Minimal, Iterate Based on Failures

**Anti-pattern**:
```
Day 1: Write 500 lines anticipating every edge case
```

**Best practice**:
```
Day 1: Write 50 lines covering core constraints
Day N: Add rule when you observe a failure pattern
```

**Example evolution**:

**Initial** (60 lines):
```markdown
# GitHub Copilot Custom Instructions

## Project Overview
FastAPI REST API for user management.

## Memory-Bank Navigation
Start with `memory-bank/INDEX.md`

## Code Standards
- Formatting: Black
- Type hints: Required
- Testing: pytest
```

**After observing SQL injection attempt** (+15 lines):
```markdown
## Critical Constraints

### Security (DO NOT)
- ❌ String concatenation in SQL (allows injection)
- ✅ Use parameterized queries
```

**After observing N+1 query bug** (+10 lines):
```markdown
### Performance (DO NOT)
- ❌ N+1 database queries
- ✅ Use eager loading/joins
```

---

### 2. Use Critical Constraints Effectively

Focus on what **breaks builds**, **causes bugs**, or **violates security**.

**Structure**:
```markdown
## Critical Constraints

### Security (DO NOT)
- ❌ [Anti-pattern] - [Why it's bad]
- ✅ [Correct approach]

### Performance (DO NOT)
- ❌ [Anti-pattern] - [Why it's bad]
- ✅ [Correct approach]

### Breaking Changes (DO NOT)
- ❌ [Anti-pattern] - [Why it's bad]
- ✅ [Correct approach]
```

**Example**:
```markdown
## Critical Constraints

### Security (DO NOT)
- ❌ SQL string concatenation (`WHERE x = '${x}'`) - Allows injection
- ❌ Hardcoded secrets in code - Security violation
- ❌ Exposing password_hash in API responses - Data leak
- ✅ Use parameterized queries (`WHERE x = ?`)
- ✅ Load secrets from environment/vault
- ✅ Exclude sensitive fields from Pydantic models

### Performance (DO NOT)
- ❌ N+1 queries (loop + query per item) - Kills performance
- ❌ Full table scans without WHERE clause - Slow queries
- ❌ Sync I/O in async endpoints - Blocks event loop
- ✅ Use eager loading/joins (query once)
- ✅ Add indexes on filter columns
- ✅ Use async operations for I/O
```

---

### 3. Reference Memory-Bank Consistently

Every section should point to memory-bank for details:

```markdown
## Architecture

### Hexagonal Design (Ports & Adapters)

Brief overview here (2-3 sentences).

**Detailed documentation**: `memory-bank/architecture/overview.md` (87 lines)

**Key patterns**:
- Domain has no dependencies on adapters
- Request flow: API → Service → Repository → Database

---

## Quick Reference

### Adding New Endpoint
See `memory-bank/workflows/adding-endpoint.md` for complete guide.

Quick steps:
1. Create Pydantic models
2. Add route
3. Implement service
4. Write tests

### Testing Changes
See `memory-bank/workflows/testing.md` for complete guide.

### Debugging Issues
See `memory-bank/workflows/troubleshooting.md` for common issues.
```

---

### 4. Avoid Generic Advice

Generic advice is already known by Copilot. Be **specific** to your project.

**Bad** (generic):
```markdown
## Best Practices

- Write clean code
- Follow best practices
- Be consistent
- Think about performance
- Handle errors properly
- Write good tests
```

**Good** (specific):
```markdown
## Code Standards

**Naming conventions**:
- Variables/functions: `snake_case`
- Classes: `PascalCase`
- Constants: `UPPER_CASE`
- Test files: `test_*.py`

**Required**:
- Type hints on all functions
- Docstrings for public APIs (Google style)
- Unit tests for business logic
- Integration tests for API endpoints
```

---

## Testing Your Instructions

### 1. Verify File Location

```bash
ls -la .github/copilot-instructions.md
```

File must be at repository root in `.github/` directory.

### 2. Test Code Generation

**Test naming conventions**:
- Create a new function - does Copilot use your naming style?
- Create a new class - does it follow your class naming?

**Test constraint adherence**:
- Write a database query - does it avoid string concatenation?
- Create an API endpoint - does it use your patterns?

**Test memory-bank awareness**:
- Ask Copilot "how do I add a new endpoint?" - does it reference memory-bank?

### 3. Measure Effectiveness

Track over 2-3 weeks:

| Metric | Target | Measurement |
|--------|--------|-------------|
| **First-attempt success** | > 70% | Tasks completed correctly first time |
| **Anti-pattern violations** | < 5% | Times Copilot suggests anti-patterns |
| **Correct pattern usage** | > 80% | Times Copilot uses documented patterns |
| **Memory-bank references** | > 50% | Times Copilot points to memory-bank |

---

## Advanced Patterns

### 1. Multi-Tool Strategy

Use GitHub Copilot alongside other tools for team flexibility:

```
project/
├── .github/
│   └── copilot-instructions.md    # GitHub Copilot
├── .cursor/rules/
│   └── workspace.mdc               # Cursor
├── .windsurf/rules/
│   └── workspace.md                # Windsurf
├── CLAUDE.md                       # Claude Code
└── memory-bank/                    # Shared documentation
    └── INDEX.md
```

**Benefits**:
- Team members can use any tool
- Content is ~95% identical (easy to maintain)
- Memory-bank is 100% shared
- Single source of truth

**Maintenance**:
- Update memory-bank once
- Sync constraints across tool files (script this)
- Test with multiple tools

---

### 2. Monorepo Configuration

**Single file at root**:
```
monorepo/
├── .github/
│   └── copilot-instructions.md    # Workspace-wide (< 200 lines)
├── project-a/
│   └── memory-bank/                # Project-specific docs
├── project-b/
│   └── memory-bank/                # Project-specific docs
└── memory-bank/                    # Workspace-wide docs
    └── INDEX.md
```

**Instructions structure**:
```markdown
# GitHub Copilot Custom Instructions

## Workspace Overview

Multi-project monorepo.

| Project | Purpose | Language |
|---------|---------|----------|
| `project-a/` | API service | Python 3.12 |
| `project-b/` | Web frontend | TypeScript/React |

## Navigation

- Workspace: `memory-bank/INDEX.md`
- Project A: `project-a/memory-bank/INDEX.md`
- Project B: `project-b/memory-bank/INDEX.md`

## Code Standards

**Both projects**:
- Format on save: Enabled
- Type annotations: Required

**Project A** (Python):
- Formatting: Black
- Testing: pytest

**Project B** (TypeScript):
- Formatting: Prettier
- Testing: Jest
```

---

### 3. File Size Annotations

Help Copilot budget context by including file sizes:

```markdown
## Memory-Bank Navigation

### File Size Reference

| Category | Lines | Token Cost | Use Case |
|----------|-------|------------|----------|
| Lightweight | < 150 | ~500 | Quick lookups |
| Medium | 150-350 | ~1,000-1,500 | Component docs |
| Large | 350-500 | ~1,500-2,000 | Workflows |

### Documentation

**Architecture** (~250 lines total):
- `memory-bank/architecture/overview.md` (87 lines) - System design
- `memory-bank/architecture/data-flow.md` (163 lines) - Request flow

**Components** (~1,200 lines total):
- `memory-bank/components/api-service.md` (263 lines) - API patterns
- `memory-bank/components/database.md` (221 lines) - DB access

**Workflows** (~800 lines total):
- `memory-bank/workflows/adding-endpoint.md` (365 lines) - New endpoints
- `memory-bank/workflows/testing.md` (360 lines) - Test strategies
```

---

## Troubleshooting

### Instructions Not Loading

**Symptoms**:
- Copilot doesn't follow your conventions
- Copilot doesn't reference memory-bank
- Copilot suggests anti-patterns

**Fixes**:
1. Verify file location: `.github/copilot-instructions.md` (not `.github/COPILOT.md` or other variations)
2. Ensure file is committed to repository
3. Restart IDE or reload window
4. Check GitHub Copilot extension is enabled and authenticated

---

### Instructions Too Generic

**Symptoms**:
- Instructions could apply to any project
- No project-specific patterns
- Copilot behavior unchanged

**Fixes**:
1. Remove generic advice ("write clean code")
2. Add specific constraints from your project
3. Include actual anti-patterns you've observed
4. Reference your memory-bank structure

**Before** (generic):
```markdown
## Best Practices
- Write clean code
- Test your code
- Use good names
```

**After** (specific):
```markdown
## Critical Constraints

### Security
- ❌ SQL string concatenation (we had injection bug)
- ✅ Use parameterized queries via SQLAlchemy

### Performance
- ❌ N+1 queries (we had 500ms → 5s degradation)
- ✅ Use eager loading (.options(joinedload()))
```

---

### Instructions Too Long

**Symptoms**:
- File > 300 lines
- Contains detailed documentation
- Duplicates memory-bank content

**Fixes**:
1. Move detailed docs to memory-bank
2. Replace documentation with pointers
3. Keep only critical constraints
4. Remove examples (link to memory-bank instead)

---

## Real-World Examples

### Example 1: FastAPI Microservice

See: `examples/simple-project/.github/copilot-instructions.md`

**Characteristics**:
- Single service
- Python + FastAPI
- Hexagonal architecture
- 150 lines total

**Key sections**:
- Quick overview (20 lines)
- Development commands (30 lines)
- Architecture brief (20 lines)
- Critical constraints (40 lines)
- Memory-bank pointers (40 lines)

---

### Example 2: Monorepo Data Platform

See: `examples/monorepo-data-platform/` (if implemented)

**Characteristics**:
- Multiple services
- Multi-language (Python, TypeScript, Go)
- Shared memory-bank
- 200 lines total

**Key sections**:
- Workspace overview (30 lines)
- Per-project standards (40 lines)
- Shared constraints (50 lines)
- Navigation (80 lines)

---

## Resources

### Documentation
- **Memory-Bank Pattern**: `docs/fundamentals/memory-bank-pattern.md` - Foundation pattern
- **Rule Writing**: `docs/best-practices/rule-writing.md` - How to write effective rules
- **Context Budgeting**: `docs/best-practices/context-budgeting.md` - Managing context limits
- **Anti-Patterns**: `docs/best-practices/anti-patterns.md` - What to avoid

### Templates
- **Copilot Template**: `templates/copilot/.github/copilot-instructions.md.template`
- **Memory-Bank Template**: `templates/memory-bank/INDEX.md.template`

### Checklists
- **Setup Checklist**: `checklists/setup-checklist.md`
- **Rule Review**: `checklists/rule-review-checklist.md`
- **Context Health**: `checklists/context-health-check.md`

---

## Summary

### Key Takeaways

1. **Single file**: `.github/copilot-instructions.md` (< 200 lines)
2. **Plain Markdown**: No YAML frontmatter needed
3. **Always loaded**: No conditional loading (unlike Cursor/Windsurf)
4. **Constraints over docs**: Focus on what not to do
5. **Memory-bank pointers**: Reference detailed docs, don't duplicate
6. **Task-based organization**: Help Copilot understand intent
7. **Start minimal**: Add rules based on observed failures
8. **Specific, not generic**: Project-specific constraints only

### Quick Setup

```bash
# 1. Create directory
mkdir -p .github

# 2. Copy template
cp templates/copilot/.github/copilot-instructions.md.template .github/copilot-instructions.md

# 3. Customize
vim .github/copilot-instructions.md

# 4. Commit
git add .github/copilot-instructions.md
git commit -m "Add GitHub Copilot custom instructions"
```

**Done!** GitHub Copilot now has effective context for your repository.
