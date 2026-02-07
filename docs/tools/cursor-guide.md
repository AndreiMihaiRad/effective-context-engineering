# Cursor IDE: Context Engineering Guide

> **Key insight**: Cursor's power comes from precise, conditional rule loading through glob patterns and the memory-bank pattern.

**Attribution**: Patterns from FanDuel nexus and data-api production implementations.

---

## Quick Start

### 1. Create Directory Structure

```bash
mkdir -p .cursor/rules
```

### 2. Create Your First Rule

Create `.cursor/rules/workspace.mdc`:

```yaml
---
description: Workspace overview
alwaysApply: true
---

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

### The .mdc File Format

Cursor rules use **MDC (Markdown with YAML frontmatter)** files:

```yaml
---
description: Human-readable description
alwaysApply: true  # OR globs: ["pattern"]
---

# Markdown Content

Your rules, constraints, and guidance here.
```

### Two Critical Fields

**1. `description`** (required)
- Human-readable summary
- Shows in Cursor's rules panel
- Keep under 100 characters

**2. Rule Activation** (choose one)

**Option A: `alwaysApply: true`**
- Active in every session
- Use for workspace-level rules
- Keep minimal (< 100 lines)

**Option B: `globs: ["pattern"]`**
- Active only when editing matching files
- Use for project/component-specific rules
- Supports standard glob patterns

---

## Rule Types and Usage

### Type 1: Always-Apply Rules (Workspace Level)

**Purpose**: Universal constraints, navigation, code standards

**Example** (`.cursor/rules/workspace.mdc`):

```yaml
---
description: Nexus workspace - FanDuel data platform libraries
alwaysApply: true
---

# Nexus Workspace

## Overview

| Package | Purpose | Python |
|---------|---------|--------|
| `fd-nexus/` | Spark library for Databricks pipelines | 3.10+ |
| `orbit-utils/` | Airflow operators | 3.12+ |

## Memory-Bank Navigation
Start with `memory-bank/INDEX.md` for workspace-wide documentation.

## Code Standards
- Formatting: Black-compatible (ruff for fd-nexus)
- Type annotations: Required for all public APIs
- Package management: hatch

Naming: `snake_case` (variables/functions), `PascalCase` (classes), `UPPER_CASE` (constants)
```

**Key points**:
- < 100 lines
- High-level overview only
- Points to memory-bank for details
- Universal constraints

### Type 2: Glob-Based Rules (Project/Component Level)

**Purpose**: Project-specific patterns, anti-patterns, architecture

**Example** (`.cursor/rules/fd-nexus.mdc`):

```yaml
---
description: fd-nexus Spark library rules - applies when editing fd-nexus files
globs: ["fd-nexus/**/*"]
---

# fd-nexus Rules

## Project Context
Enterprise-grade Python library for Apache Spark on Databricks. 
Three-layer architecture: Script → Component → Utility.

## Key Patterns

### Script Pattern
```python
BaseJob → BaseIngestScript → SpecificIngestScript
```

### Configuration Pattern
```python
class XReaderConfig:
    @staticmethod
    def define_args(parser: ArgumentParser) -> None
    @classmethod
    def from_args(cls, args: Namespace) -> "XReaderConfig"
```

### Transformation Registry
50+ pre-built transformations in `fd-nexus/src/fd_nexus/spark/transforms/registry.py`

## Memory-Bank First
**ALWAYS consult** `memory-bank/fd-nexus/INDEX.md` before implementation

## Anti-Patterns
- NEVER override `run()` in BaseJob
- NEVER use argparse directly (use typed config classes)
- ALWAYS check 50+ existing transformations before creating new ones
```

**Key points**:
- Activated by file pattern
- Project-specific constraints
- Concrete anti-patterns
- References memory-bank for details

---

## Glob Pattern Guide

### Common Patterns

```yaml
# Single project
globs: ["project-name/**/*"]

# Multiple projects
globs: ["project-a/**/*", "project-b/**/*"]

# Specific file types
globs: ["**/*.py", "**/*.ts"]

# Exclude patterns (use negation in separate rule)
globs: ["**/*", "!**/node_modules/**/*"]

# Specific directories
globs: ["src/components/**/*", "tests/unit/**/*"]
```

### Pattern Priority

When multiple rules match:
1. More specific patterns take precedence
2. Rules are merged (not replaced)
3. Later rules can override earlier ones

**Example**:
```yaml
# workspace.mdc - Always applies
alwaysApply: true

# api.mdc - Adds when editing API files
globs: ["src/api/**/*"]

# auth.mdc - Further specializes for auth files
globs: ["src/api/auth/**/*"]
```

Result when editing `src/api/auth/login.ts`:
- workspace.mdc (always)
- api.mdc (matches src/api/**/*)
- auth.mdc (matches src/api/auth/**/*)

---

## Best Practices

### 1. Keep Each Rule Under 100 Lines

```
❌ Single 500-line .cursorrules file
✅ Multiple focused files:
   - workspace.mdc (44 lines)
   - project-a.mdc (60 lines)
   - project-b.mdc (55 lines)
```

### 2. Split by Concern, Not by Size

```
❌ rules-part1.mdc, rules-part2.mdc
✅ workspace.mdc, frontend.mdc, backend.mdc
```

### 3. Use Specific, Actionable Rules

```yaml
❌ "Write clean code"
❌ "Follow best practices"

✅ "Never override run() in BaseJob"
✅ "Use Delta Lake for transactional writes"
✅ "Check 50+ existing transformations before creating new ones"
```

### 4. Reference Memory-Bank for Details

```yaml
---
description: Project rules
globs: ["project/**/*"]
---

# Project Rules

## Memory-Bank First
**ALWAYS consult** `memory-bank/project/INDEX.md` before implementation

## Core Constraints
[Keep this section under 50 lines]

## Anti-Patterns
[Specific things to avoid]
```

### 5. One Always-Apply Rule

```
✅ workspace.mdc (alwaysApply: true)
❌ rule1.mdc (alwaysApply: true)
❌ rule2.mdc (alwaysApply: true)  # Too many!
```

Most projects need only **one** always-apply rule for workspace basics.

### 6. Version Control All Rules

```bash
# Commit
.cursor/rules/*.mdc
memory-bank/

# .gitignore (user-level only)
.cursor/settings.json
```

---

## Integration with Memory-Bank Pattern

### The Pattern

**Rules** = Constraints + navigation pointers (< 100 lines)
**Memory-bank** = Documentation (any size)

### Example Implementation

**.cursor/rules/workspace.mdc** (44 lines):
```yaml
---
description: Workspace overview
alwaysApply: true
---

# Workspace Rules

## Memory-Bank Navigation
Start with `memory-bank/INDEX.md` for documentation.

## Code Standards
- Formatting: Black-compatible
- Type annotations: Required
```

**.cursor/rules/project.mdc** (60 lines):
```yaml
---
description: Project-specific rules
globs: ["project/**/*"]
---

# Project Rules

## Memory-Bank First
**ALWAYS consult** `memory-bank/project/INDEX.md` before implementation

## Key Patterns
[List 3-5 most important patterns]

## Anti-Patterns
[List 3-5 most important anti-patterns]
```

**memory-bank/INDEX.md** (< 100 lines):
```markdown
# Memory-Bank Index

## Navigation by Task
| Task | Start With | Size |
|------|------------|------|
| [Task] | `path/to/doc.md` | [N lines] |

## File Size Reference
[Categorized list with line counts]
```

**Result**: Rules stay tiny, AI fetches docs just-in-time.

---

## Common Pitfalls

### 1. Too Many Rules

```
❌ 15 .mdc files for a single project
✅ 3-5 .mdc files total
```

More rules = slower loading, potential conflicts.

### 2. Embedding Documentation in Rules

```
❌ .cursor/rules/api.mdc (1,200 lines of API docs)
✅ .cursor/rules/api.mdc (60 lines) → memory-bank/api/
```

### 3. Generic Advice

```
❌ "Write tests"
❌ "Use TypeScript"
✅ "Add tests in tests/unit/ using pytest fixtures from conftest.py"
```

### 4. Duplicate Content

```
❌ Same transformations list in:
   - workspace.mdc
   - project.mdc
   - memory-bank/

✅ Single source in memory-bank/, reference from rules
```

### 5. Overly Broad Globs

```
❌ globs: ["**/*"]  # Matches everything!
✅ globs: ["src/project/**/*"]
```

Overly broad patterns defeat the purpose of conditional loading.

---

## Testing Your Rules

### 1. Check Active Rules

- Open Cursor
- Look for the rules icon (bottom right)
- Click to see which rules are currently active
- Verify expected rules are loaded

### 2. Test Glob Patterns

Create test files:
```bash
touch project-a/test.py
touch project-b/test.py
```

Edit each file, check which rules activate.

### 3. Verify Memory-Bank References

Ask Cursor: "What documentation is available for X?"

Expected: References memory-bank, reads INDEX.md, finds relevant docs.

### 4. Test Context Budget

Ask for a complex task and monitor:
- Does it read entire files or sections?
- Does it reference memory-bank appropriately?
- Is context usage reasonable?

---

## Example Configurations

### Minimal Single-Project Setup

```
.cursor/rules/
└── project.mdc (< 100 lines)
```

```yaml
---
description: Project rules
alwaysApply: true
---

# Project Rules

## Memory-Bank
`memory-bank/INDEX.md` - Start here for all documentation

## Tech Stack
- Language: TypeScript
- Framework: React + Next.js
- Testing: Jest + React Testing Library

## Code Standards
- Formatting: Prettier
- Type safety: strict mode
- Component style: Functional with hooks
```

### Monorepo Setup

```
.cursor/rules/
├── workspace.mdc (< 100 lines, alwaysApply: true)
├── frontend.mdc (< 100 lines, globs: ["frontend/**/*"])
├── backend.mdc (< 100 lines, globs: ["backend/**/*"])
└── shared.mdc (< 100 lines, globs: ["shared/**/*"])
```

**workspace.mdc**:
- Workspace overview
- Universal code standards
- Memory-bank navigation

**Project-specific .mdc files**:
- Project context
- Key patterns
- Anti-patterns
- Memory-bank reference for project

---

## Migration from .cursorrules

If you have an existing `.cursorrules` file:

### Step 1: Create Directory

```bash
mkdir -p .cursor/rules
```

### Step 2: Split Content

Extract sections into focused files:
- Workspace basics → `workspace.mdc` (alwaysApply: true)
- Project-specific → `project.mdc` (globs pattern)
- Documentation → Move to `memory-bank/`

### Step 3: Add Frontmatter

```yaml
---
description: [Description]
alwaysApply: true  # or globs: [...]
---

[Your content]
```

### Step 4: Delete Old File

```bash
rm .cursorrules  # or .cursor/rules
```

The new `.cursor/rules/*.mdc` format takes precedence.

---

## Summary

| Aspect | Best Practice |
|--------|---------------|
| **File location** | `.cursor/rules/*.mdc` |
| **Format** | YAML frontmatter + Markdown |
| **Rule types** | `alwaysApply` or `globs` |
| **Rule size** | < 100 lines each |
| **Content** | Constraints + memory-bank references |
| **Number of rules** | 3-5 total for most projects |
| **Documentation** | External memory-bank, not in rules |

---

## References

- [Cursor Rules Documentation](https://docs.cursor.com/context/rules)
- See `docs/fundamentals/memory-bank-pattern.md` for the memory-bank pattern
- See `templates/cursor/` for ready-to-use templates
- See `examples/` for real-world configurations
