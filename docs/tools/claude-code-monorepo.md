# Claude Code: Monorepo Guide

> **Key insight**: Claude Code's CLAUDE.md loading follows specific directory semantics -- understanding them prevents silent configuration failures in monorepos.

**Attribution**: Synthesized from Anthropic documentation and production monorepo implementations.

---

## CLAUDE.md Loading Semantics

Claude Code follows three loading behaviors based on directory relationship to cwd:

### 1. Ancestor (UP)

At startup, Claude Code walks **up** from cwd, loading every CLAUDE.md it finds. The root CLAUDE.md always loads regardless of where you launch Claude Code.

### 2. Descendant (DOWN)

When Claude touches a file in a subdirectory, it **lazy-loads** that directory's CLAUDE.md on demand. Not eager -- only triggers on actual file access.

### 3. Siblings (NEVER)

CLAUDE.md files in sibling directories **never** load unless Claude actively works there. Working in `frontend/` does not load `backend/CLAUDE.md`.

### Complete Example

```
monorepo/
├── CLAUDE.md                    # Always loads (ancestor)
├── packages/
│   ├── CLAUDE.md               # Loads when working in packages/
│   ├── frontend/
│   │   └── CLAUDE.md           # Lazy-loads when touching frontend/ files
│   ├── backend/
│   │   └── CLAUDE.md           # Lazy-loads when touching backend/ files
│   └── shared/
│       └── CLAUDE.md           # NEVER loads when working in frontend/
└── tools/
    └── CLAUDE.md               # NEVER loads when working in packages/
```

**Scenario**: Run `claude` from `monorepo/`, then refactor a backend endpoint:

1. `monorepo/CLAUDE.md` loaded at startup (ancestor)
2. Claude reads `packages/backend/src/api.py`
3. `packages/CLAUDE.md` lazy-loads (ancestor of backend/)
4. `packages/backend/CLAUDE.md` lazy-loads (direct parent)
5. `packages/frontend/CLAUDE.md` -- never loads (sibling, untouched)

---

## Skills Discovery in Monorepos

Skills behave differently from CLAUDE.md:

- Discovered from `.claude/skills/` at **project root only**
- Do **not** walk ancestors or support per-package directories
- Per-package `.claude/skills/` directories are **silently ignored**

**Workaround**: Use `$ARGUMENTS` for package-specific skills:

```markdown
<!-- .claude/skills/test-package.md -->
Run the test suite for the specified package.
Package: $ARGUMENTS
Steps:
1. cd to packages/$ARGUMENTS
2. Run the package-specific test command
```

Usage: `/test-package frontend`

---

## Best Practices for CLAUDE.md Hierarchy

### Root CLAUDE.md (Always Loaded)

Loads for **every** task, so budget is tight. **Target: < 50 lines.**

**Include**: workspace description, package table, shared standards, memory-bank pointer.
**Exclude**: package-specific commands, detailed architecture, long constraint lists.

**Example** (35 lines):

```markdown
# CLAUDE.md

Monorepo for the Acme data platform.

## Packages

| Package | Purpose | Stack |
|---------|---------|-------|
| `packages/api` | REST API | Python 3.12, FastAPI |
| `packages/web` | Dashboard | TypeScript, React |
| `packages/shared` | Shared utils | Python 3.12 |

## Shared Standards
- Formatting: Black (Python), Prettier (TS)
- Commits: conventional commits (feat:, fix:, chore:)

## Navigation
Each package has its own CLAUDE.md (lazy-loaded when working there).
Start with `memory-bank/INDEX.md` for detailed documentation.
```

### Package CLAUDE.md (Lazy Loaded)

Loads only when Claude works in that package. **Target: < 100 lines each.**

**Include**: dev commands, package architecture, package constraints, key file paths.

**Example** (`packages/api/CLAUDE.md`):

```markdown
# API Package

FastAPI REST service for the Acme data platform.

## Commands
cd packages/api
poetry install          # Setup
poetry run pytest       # Tests
poetry run uvicorn main:app --reload  # Dev server

## Architecture
Hexagonal: src/domain/ (logic), src/adapters/ (routes, DB), src/ports/ (interfaces)

## Constraints
- Never add sync DB calls in async endpoints
- Always write unit + integration tests for new endpoints

## Docs
See `memory-bank/api/` for detailed documentation.
```

### When NOT to Use Subdirectory CLAUDE.md

- **Small monorepos (< 3 packages)**: single root CLAUDE.md under 150 lines suffices
- **Homogeneous tech stack**: little package-specific guidance to give
- **Maintenance overhead**: fewer files is better if the team rarely updates them

---

## Example Directory Structures

### Small Monorepo (2-3 packages)

```
monorepo/
├── CLAUDE.md                    # Everything in one file (< 120 lines)
├── .claude/skills/
├── memory-bank/INDEX.md
└── packages/{api,web}/
```

### Medium Monorepo (4-8 packages)

```
monorepo/
├── CLAUDE.md                    # Workspace overview (< 50 lines)
├── .claude/skills/
├── memory-bank/{INDEX.md,api/,web/}
└── packages/
    ├── api/CLAUDE.md            # Per-package (< 100 lines each)
    ├── web/CLAUDE.md
    ├── shared/CLAUDE.md
    └── workers/CLAUDE.md
```

### Large Monorepo (10+ packages)

```
monorepo/
├── CLAUDE.md                    # Workspace overview (< 40 lines)
├── .claude/skills/
├── memory-bank/{INDEX.md,...}
├── services/
│   ├── CLAUDE.md               # Category patterns (< 60 lines)
│   ├── auth/CLAUDE.md
│   └── billing/CLAUDE.md
├── libs/
│   ├── CLAUDE.md               # Library patterns (< 60 lines)
│   └── ui-components/CLAUDE.md
└── tools/
    ├── CLAUDE.md
    └── cli/CLAUDE.md
```

Category-level CLAUDE.md (e.g., `services/CLAUDE.md`) captures patterns shared across all services but not applicable to libs or tools.

---

## Integration with Memory-Bank

The memory-bank pattern scales alongside the CLAUDE.md hierarchy in monorepos.

```
memory-bank/
├── INDEX.md                # Workspace navigation, routes to package indexes
├── cross-cutting/          # Auth, logging, CI/CD (spans packages)
│   ├── authentication.md
│   └── ci-cd.md
├── api/
│   ├── INDEX.md            # API-specific navigation
│   └── endpoints.md
└── web/
    ├── INDEX.md            # Web-specific navigation
    └── components.md
```

Root `INDEX.md` routes to package-specific sub-indexes. Cross-cutting docs live at root level since they span multiple packages.

---

## Common Pitfalls

1. **Root CLAUDE.md too large** -- Root loads for every task. Move package content into package CLAUDE.md files where it lazy-loads on demand.
2. **Assuming sibling CLAUDE.md files load** -- `packages/shared/CLAUDE.md` is invisible when working in `packages/api/`. Anything always needed goes in root or ancestor CLAUDE.md.
3. **Skills in package directories** -- Per-package `.claude/skills/` are silently ignored. Centralize at root and use `$ARGUMENTS`.
4. **Not using lazy loading** -- Stuffing everything in root "just in case" defeats lazy loading. Trust the mechanism.
5. **Inconsistent hierarchy** -- Some packages with CLAUDE.md, some without, creates guidance gaps. Apply one strategy consistently.

---

## References

- See `docs/tools/claude-code-guide.md` for the general Claude Code context engineering guide
- See `docs/fundamentals/memory-bank-pattern.md` for the memory-bank pattern in detail
- See `docs/best-practices/context-budgeting.md` for context window management
- See `examples/monorepo-data-platform/` for a production monorepo example
- [Using CLAUDE.md Files](https://www.claude.com/blog/using-claude-md-files) -- Anthropic documentation
- [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices) -- Anthropic engineering blog
