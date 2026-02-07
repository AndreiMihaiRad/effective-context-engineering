# Cursor Templates

Ready-to-use templates for Cursor IDE context configuration.

## Files Included

- `.cursor/rules/workspace.mdc.template` - Workspace-level rules (always apply)
- `.cursor/rules/project.mdc.template` - Project-specific rules (glob-based)

## Quick Start

### 1. Copy Templates

```bash
# From your project root
mkdir -p .cursor/rules
cp path/to/templates/cursor/.cursor/rules/*.template .cursor/rules/
```

### 2. Rename Files

```bash
cd .cursor/rules
mv workspace.mdc.template workspace.mdc
mv project.mdc.template my-project.mdc  # Name after your project
```

### 3. Fill in Placeholders

Replace all `[PLACEHOLDER]` values:

**workspace.mdc**:
- `[PROJECT_NAME]` - Your workspace/monorepo name
- `[package-1]`, `[package-2]` - Your package names
- `[Purpose]` - What each package does
- `[Language + version]` - e.g., "Python 3.12", "TypeScript 5"
- `[tool]` - Formatting tool, package manager, etc.
- `[case]` - Naming conventions (snake_case, camelCase, etc.)

**project.mdc**:
- `[PROJECT_NAME]` - Project name
- `[project-dir]` - Directory path for glob pattern
- `[language]` - Programming language
- Fill in patterns, anti-patterns, commands

### 4. Create Memory-Bank

```bash
mkdir -p memory-bank
cp path/to/templates/memory-bank/INDEX.md.template memory-bank/INDEX.md
# Edit memory-bank/INDEX.md with your documentation structure
```

## Template Details

### workspace.mdc.template

**Purpose**: Always-active rules for entire workspace

**Key sections**:
- Overview with package table
- Memory-bank navigation
- Code standards
- Quick reference

**Target size**: < 100 lines

**YAML frontmatter**:
```yaml
---
description: [PROJECT_NAME] workspace overview
alwaysApply: true
---
```

### project.mdc.template

**Purpose**: Project-specific rules activated by file patterns

**Key sections**:
- Project context
- Key patterns
- Memory-bank references
- Development commands
- Anti-patterns

**Target size**: < 100 lines

**YAML frontmatter**:
```yaml
---
description: [PROJECT_NAME] rules - applies when editing [PROJECT] files
globs: ["[project-dir]/**/*"]
---
```

**Important**: The `globs` field determines when this rule activates.

## Best Practices

### 1. Keep Rules Minimal

- Move detailed documentation to `memory-bank/`
- Reference memory-bank from rules
- Target < 100 lines per file

### 2. Be Specific

Avoid:
```markdown
- Write clean code
- Follow best practices
```

Prefer:
```markdown
- Never override run() in BaseJob
- Always check 50+ existing transformations before creating new ones
```

### 3. Use Clear Glob Patterns

```yaml
# Specific to API code
globs: ["src/api/**/*"]

# Multiple related directories
globs: ["frontend/src/**/*", "frontend/tests/**/*"]

# Specific file types
globs: ["**/*.py"]

# Avoid: Too broad
globs: ["**/*"]  # Matches everything!
```

### 4. Reference Memory-Bank

```markdown
## Memory-Bank First

**ALWAYS consult** `memory-bank/project/INDEX.md` before implementation.
```

## Example: Single-Project Setup

**Minimal setup for a single-project repository**:

```
project/
├── .cursor/rules/
│   └── project.mdc (use alwaysApply: true)
└── memory-bank/
    └── INDEX.md
```

**project.mdc**:
```yaml
---
description: Project rules
alwaysApply: true  # Since it's the only project
---

# Project Rules

## Memory-Bank
See `memory-bank/INDEX.md`

[... rest of rules]
```

## Example: Monorepo Setup

**Setup for multiple projects in one repository**:

```
monorepo/
├── .cursor/rules/
│   ├── workspace.mdc (alwaysApply: true)
│   ├── frontend.mdc (globs: ["frontend/**/*"])
│   └── backend.mdc (globs: ["backend/**/*"])
└── memory-bank/
    ├── INDEX.md
    ├── frontend/
    └── backend/
```

## Testing Your Configuration

1. **Open Cursor** in your project
2. **Check active rules**: Look for rules icon (bottom right)
3. **Edit a file**: Verify appropriate rules load
4. **Test memory-bank**: Ask "What documentation is available?"
5. **Verify behavior**: Ensure AI follows patterns and anti-patterns

## Common Issues

### Rules Not Loading

**Check**:
- Files are in `.cursor/rules/` directory
- Files have `.mdc` extension (not `.mdc.template`)
- YAML frontmatter is valid
- No syntax errors

### Wrong Rules Active

**Check**:
- Glob patterns match intended files
- No typos in glob patterns
- Test with: Edit file, check which rules load

### Rules Too Long

**Fix**:
- Move details to `memory-bank/`
- Keep only constraints in rules
- Split into multiple focused files

## Next Steps

1. Fill in template placeholders
2. Create `memory-bank/INDEX.md` from template
3. Test configuration with real tasks
4. Iterate based on AI behavior
5. Add project-specific docs to `memory-bank/`

## References

- See `docs/tools/cursor-guide.md` for comprehensive guide
- See `docs/fundamentals/memory-bank-pattern.md` for memory-bank details
- See `docs/best-practices/rule-writing.md` for writing effective rules
- See `examples/` for real-world configurations
