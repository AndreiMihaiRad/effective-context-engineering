# Windsurf IDE: Context Engineering Guide

> **Key insight**: Windsurf uses the same strategy as Cursor—only the format differs. Keep content 95% identical.

**Attribution**: Patterns adapted from Cursor implementations (FanDuel nexus, data-api).

---

## Quick Start

### 1. Create Directory Structure

```bash
mkdir -p .windsurf/rules
```

### 2. Create Your First Rule

Create `.windsurf/rules/workspace.md`:

```markdown
# Workspace Rules

## Memory-Bank Navigation
Start with `memory-bank/INDEX.md` for detailed documentation.

## Code Standards
- Formatting: [tool]
- Type annotations: [required/optional]
- Package management: [tool]
```

### 3. Test It

Open any file in your project - the rule should be active.

---

## Configuration Format

### The .md File Format

Windsurf rules use **plain Markdown files** (no YAML frontmatter):

```markdown
# Title

Your rules, constraints, and guidance here.
```

**Key difference from Cursor**:
- No YAML frontmatter (`---`)
- No `alwaysApply` or `globs` metadata
- Just Markdown content

---

## Differences from Cursor

| Feature | Cursor | Windsurf |
|---------|--------|----------|
| **File extension** | `.mdc` | `.md` |
| **YAML frontmatter** | Yes (`---`) | No |
| **Conditional loading** | `globs: ["pattern"]` | Not supported |
| **Always-apply** | `alwaysApply: true` | All rules always apply |
| **Content** | 95% same | 95% same |

### What This Means

**Cursor** can conditionally load rules based on file patterns:
```yaml
---
globs: ["frontend/**/*"]  # Only when editing frontend
---
```

**Windsurf** loads all rules in `.windsurf/rules/` for every session:
```markdown
# All rules always active
```

**Implication**: Keep Windsurf rules even more minimal since they're always loaded.

---

## Rule Organization

### Single-Project Setup

```
.windsurf/rules/
└── project.md (< 100 lines)
```

**project.md**:
```markdown
# Project Rules

## Memory-Bank Navigation
Start with `memory-bank/INDEX.md` for documentation.

## Tech Stack
- Language: TypeScript
- Framework: React + Next.js
- Testing: Jest

## Code Standards
- Formatting: Prettier
- Type safety: strict mode
```

### Monorepo/Multi-Project Setup

```
.windsurf/rules/
├── workspace.md (< 50 lines)
├── frontend.md (< 100 lines)
├── backend.md (< 100 lines)
└── shared.md (< 100 lines)
```

**workspace.md**:
```markdown
# Workspace Overview

## Projects
| Package | Purpose | Language |
|---------|---------|----------|
| `frontend/` | React app | TypeScript |
| `backend/` | API server | Python |

## Memory-Bank Navigation
Start with `memory-bank/INDEX.md`.

## Universal Standards
- Use conventional commits
- Write tests before implementation
```

**frontend.md**:
```markdown
# Frontend Rules

**Note**: These rules apply to frontend code. Check file paths before applying.

## Memory-Bank
`memory-bank/frontend/INDEX.md` - Frontend documentation

## Key Patterns
- Functional components with hooks
- React Query for data fetching
- Zod for validation

## Anti-Patterns
- No class components
- No prop drilling (use context)
```

**Important**: Since Windsurf can't conditionally load, each rule should:
1. State its scope clearly ("Frontend Rules", "Backend Rules")
2. Be kept minimal (< 100 lines)
3. Use clear headings to aid skimming

---

## Maintaining Consistency with Cursor

### The 95% Rule

Content should be **95% identical** between Cursor and Windsurf:

**Cursor** (`.cursor/rules/project.mdc`):
```yaml
---
description: Project rules
globs: ["project/**/*"]
---

# Project Rules

## Memory-Bank First
**ALWAYS consult** `memory-bank/project/INDEX.md`

## Key Patterns
- Pattern 1
- Pattern 2
```

**Windsurf** (`.windsurf/rules/project.md`):
```markdown
# Project Rules

**Scope**: Applies to `project/**/*` files

## Memory-Bank First
**ALWAYS consult** `memory-bank/project/INDEX.md`

## Key Patterns
- Pattern 1
- Pattern 2
```

**Differences**:
- No YAML frontmatter (Windsurf)
- Explicit scope note (Windsurf, since no globs)
- Otherwise identical content

### Conversion Script Example

```bash
#!/bin/bash
# Convert Cursor .mdc to Windsurf .md

for file in .cursor/rules/*.mdc; do
    basename=$(basename "$file" .mdc)
    
    # Remove YAML frontmatter (lines between --- markers)
    sed '/^---$/,/^---$/d' "$file" > ".windsurf/rules/${basename}.md"
    
    echo "Converted $file to .windsurf/rules/${basename}.md"
done
```

**Note**: This is a starting point. Review and add scope notes manually.

---

## Best Practices

### 1. Keep Rules Even Smaller

Since all Windsurf rules load for every session:

```
Cursor target: < 100 lines per rule
Windsurf target: < 75 lines per rule
```

### 2. Clear Scope Headers

```markdown
# Frontend Rules

**Scope**: `src/frontend/**/*`
**When to apply**: Editing React components, hooks, pages

## [Rest of rules]
```

### 3. One Rule Per Concern

```
❌ single rules.md (300 lines)
✅ workspace.md (40 lines)
✅ frontend.md (70 lines)
✅ backend.md (65 lines)
```

### 4. Reference Memory-Bank Heavily

```markdown
# Backend Rules

## Memory-Bank First
For detailed API documentation, see `memory-bank/backend/INDEX.md`

## Core Constraints
[Keep this section under 30 lines]

## Anti-Patterns
[Keep this section under 30 lines]
```

Even more important in Windsurf since rules are always loaded.

### 5. Use Clear Section Headers

Since AI can't rely on glob filtering, clear headers help:

```markdown
# Database Module Rules

## Applicable To
- `src/db/**/*.py`
- `migrations/**/*.py`

## Memory-Bank
`memory-bank/database/INDEX.md`

## Constraints
[...]
```

---

## Integration with Memory-Bank Pattern

The memory-bank pattern is **even more critical** for Windsurf:

### Why?

Cursor can conditionally load rules, so you can have larger rule sets.

Windsurf loads everything, so keeping rules minimal is essential.

### Implementation

**.windsurf/rules/workspace.md** (40 lines):
```markdown
# Workspace Rules

## Memory-Bank Navigation
Start with `memory-bank/INDEX.md` for all documentation.

## Projects
- `frontend/` - React application
- `backend/` - FastAPI server

## Code Standards
- Formatting: Prettier (frontend), Black (backend)
- Type annotations: Required
```

**.windsurf/rules/backend.md** (65 lines):
```markdown
# Backend Rules

**Scope**: `backend/**/*.py`

## Memory-Bank First
**ALWAYS consult** `memory-bank/backend/INDEX.md` before implementation.

## Key Patterns
[3-5 most important patterns]

## Anti-Patterns
[3-5 most important anti-patterns]
```

**memory-bank/** (unchanged from Cursor):
- Same structure
- Same content
- Same line counts
- Tool-agnostic

---

## Common Pitfalls

### 1. Forgetting Scope Notes

```
❌ # API Rules
   [no scope indication]

✅ # API Rules
   **Scope**: `src/api/**/*.ts`
```

### 2. Too Many Rules

```
❌ 10 .md files in .windsurf/rules/
✅ 3-4 .md files maximum
```

Remember: All load for every session.

### 3. Large Rule Files

```
❌ backend.md (200 lines)
✅ backend.md (65 lines) + memory-bank references
```

### 4. Duplicating Cursor Content Exactly

```
❌ Copy .mdc file verbatim (includes YAML)
✅ Strip YAML, add scope notes, keep content
```

### 5. Not Using Memory-Bank

```
❌ Embedding all docs in rules (can't use conditional loading)
✅ Minimal rules + extensive memory-bank
```

---

## Testing Your Rules

### 1. Check Loaded Rules

- Open Windsurf
- Look for rules indicator
- Verify all `.windsurf/rules/*.md` files are loaded

### 2. Verify Memory-Bank Integration

Ask Windsurf: "What documentation is available for X?"

Expected: References memory-bank, reads INDEX.md, finds relevant docs.

### 3. Check Scope Understanding

Ask: "What are the frontend rules?"

Expected: Should reference frontend.md content and note the scope.

### 4. Monitor Context Usage

Complex task → Check if:
- Rules are appropriately sized
- Memory-bank is being used for details
- Context budget is reasonable

---

## Example Configurations

### Minimal Setup (Single Project)

**.windsurf/rules/project.md**:
```markdown
# Project Rules

## Memory-Bank
Start with `memory-bank/INDEX.md` for all documentation.

## Tech Stack
- Python 3.12
- FastAPI
- SQLAlchemy + Alembic
- pytest

## Code Standards
- Formatting: Black + isort
- Type hints: Required for all functions
- Docstrings: Google style

## Key Patterns
- Use Pydantic models for request/response
- Dependency injection for database sessions
- Async endpoints for I/O operations

## Anti-Patterns
- No sync database calls in async endpoints
- No circular imports between modules
```

### Monorepo Setup

**.windsurf/rules/workspace.md** (minimal):
```markdown
# Workspace Overview

## Projects
| Package | Purpose | Tech |
|---------|---------|------|
| `api/` | REST API | Python/FastAPI |
| `web/` | Web app | TypeScript/React |
| `shared/` | Shared types | TypeScript |

## Memory-Bank
Start with `memory-bank/INDEX.md` for workspace documentation.

## Universal Standards
- Conventional commits
- Tests required for all changes
- Type safety enforced
```

**.windsurf/rules/api.md**:
```markdown
# API Rules

**Scope**: `api/**/*.py`

## Memory-Bank
`memory-bank/api/INDEX.md` - API-specific documentation

## Architecture
FastAPI with hexagonal architecture:
- `domain/` - Business logic
- `adapters/` - External integrations
- `ports/` - Interface definitions

## Anti-Patterns
- No business logic in route handlers
- No direct database access (use repository pattern)
```

**.windsurf/rules/web.md**:
```markdown
# Web App Rules

**Scope**: `web/src/**/*.{ts,tsx}`

## Memory-Bank
`memory-bank/web/INDEX.md` - Frontend documentation

## Component Patterns
- Functional components with hooks
- React Query for server state
- Zustand for client state
- Zod for validation

## Anti-Patterns
- No prop drilling (use context or Zustand)
- No useEffect for data fetching (use React Query)
```

---

## Migration from Cursor

### Quick Migration

1. **Copy `.cursor/rules/*.mdc` to `.windsurf/rules/*.md`**

2. **Remove YAML frontmatter** from each file:
   ```bash
   # For each .md file, remove lines between --- markers
   sed -i '/^---$/,/^---$/d' .windsurf/rules/*.md
   ```

3. **Add scope notes** to project-specific rules:
   ```markdown
   # Project Rules
   
   **Scope**: `project/**/*` files
   **When to apply**: When editing project code
   
   [Rest of content...]
   ```

4. **Verify memory-bank references** are intact

5. **Test** - Open Windsurf and verify rules load correctly

### Side-by-Side Maintenance

Most teams maintain both:

```
.cursor/rules/       # Cursor format (.mdc)
.windsurf/rules/     # Windsurf format (.md)
memory-bank/         # Shared by both
```

**Strategy**:
- Edit Cursor rules normally
- Copy to Windsurf, strip frontmatter, add scope notes
- Keep memory-bank identical

**Script example**:
```bash
#!/bin/bash
# sync-rules.sh

for file in .cursor/rules/*.mdc; do
    name=$(basename "$file" .mdc)
    sed '/^---$/,/^---$/d' "$file" > ".windsurf/rules/${name}.md"
done

echo "Synced Cursor rules to Windsurf"
```

---

## Summary

| Aspect | Windsurf |
|--------|----------|
| **File location** | `.windsurf/rules/*.md` |
| **Format** | Plain Markdown (no YAML) |
| **Conditional loading** | Not supported (all rules load) |
| **Rule size** | < 75 lines each (smaller than Cursor) |
| **Content** | 95% same as Cursor rules |
| **Scope indication** | Manual notes in content |
| **Memory-bank** | Critical for keeping rules small |

---

## References

- [Windsurf Documentation](https://docs.windsurf.com/)
- [Windsurf AI Rules Guide](https://uibakery.io/blog/windsurf-ai-rules)
- See `docs/tools/cursor-guide.md` for comparison
- See `docs/fundamentals/memory-bank-pattern.md` for the memory-bank pattern
- See `templates/windsurf/` for ready-to-use templates
- See `examples/` for real-world configurations
