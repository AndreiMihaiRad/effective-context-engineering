# Windsurf Templates

Ready-to-use templates for Windsurf IDE context configuration.

## Files Included

- `.windsurf/rules/workspace.md.template` - Workspace-level rules
- `.windsurf/rules/project.md.template` - Project-specific rules

## Quick Start

### 1. Copy Templates

```bash
# From your project root
mkdir -p .windsurf/rules
cp path/to/templates/windsurf/.windsurf/rules/*.template .windsurf/rules/
```

### 2. Rename Files

```bash
cd .windsurf/rules
mv workspace.md.template workspace.md
mv project.md.template my-project.md  # Name after your project
```

### 3. Fill in Placeholders

Replace all `[PLACEHOLDER]` values:

**workspace.md**:
- `[PROJECT_NAME]` - Your workspace/monorepo name
- `[package-1]`, `[package-2]` - Your package names
- `[Purpose]` - What each package does
- `[Language + version]` - e.g., "Python 3.12", "TypeScript 5"
- `[tool]` - Formatting tool, package manager, etc.
- `[case]` - Naming conventions (snake_case, camelCase, etc.)

**project.md**:
- `[PROJECT_NAME]` - Project name
- `[project-dir]` - Directory path (for scope note)
- `[language]` - Programming language
- Fill in patterns, anti-patterns, commands

### 4. Create Memory-Bank

```bash
mkdir -p memory-bank
cp path/to/templates/memory-bank/INDEX.md.template memory-bank/INDEX.md
# Edit memory-bank/INDEX.md with your documentation structure
```

## Key Differences from Cursor

**Windsurf rules**:
- Plain Markdown (no YAML frontmatter)
- No conditional loading (all rules load always)
- Should be even more concise (< 75 lines)
- Add scope notes manually

**Cursor rules**:
- YAML frontmatter with `globs` or `alwaysApply`
- Conditional loading via glob patterns
- Target < 100 lines

## Template Details

### workspace.md.template

**Purpose**: Workspace-wide rules (always loaded)

**Key sections**:
- Overview with package table
- Memory-bank navigation
- Code standards
- Quick reference

**Target size**: < 75 lines (smaller than Cursor since always loaded)

**Format**: Plain Markdown (no frontmatter)

### project.md.template

**Purpose**: Project-specific rules (always loaded, but with scope note)

**Key sections**:
- Scope note (which files this applies to)
- Project context
- Key patterns
- Memory-bank references
- Development commands
- Anti-patterns

**Target size**: < 75 lines

**Format**: Plain Markdown with scope note

## Best Practices

### 1. Keep Rules Even Smaller

Since all rules load for every session:

- **Target**: < 75 lines per file
- **Maximum**: 3-4 rule files total
- Move even more to `memory-bank/` than Cursor

### 2. Add Clear Scope Notes

Since no glob-based loading:

```markdown
# Backend Rules

**Scope**: `backend/**/*.py` files
**When to apply**: Editing Python backend code

[... rest of rules]
```

### 3. Minimize Rule Files

```
❌ Too many (all load for every session):
.windsurf/rules/
├── workspace.md
├── frontend.md
├── backend.md
├── api.md
├── database.md
├── auth.md
└── [7+ files]

✅ Minimal (3-4 total):
.windsurf/rules/
├── workspace.md (40 lines)
├── frontend.md (70 lines)
└── backend.md (65 lines)
```

### 4. Reference Memory-Bank Heavily

```markdown
## Memory-Bank First

For detailed API documentation, see `memory-bank/api/INDEX.md`

[Keep rules section under 30 lines]
```

## Example: Single-Project Setup

**Minimal setup for a single-project repository**:

```
project/
├── .windsurf/rules/
│   └── project.md
└── memory-bank/
    └── INDEX.md
```

**project.md** (< 75 lines):
```markdown
# Project Rules

## Memory-Bank
See `memory-bank/INDEX.md` for all documentation

## Tech Stack
- Python 3.12
- FastAPI
- pytest

## Key Patterns
[3-5 most important patterns]

## Anti-Patterns
[3-5 things to avoid]
```

## Example: Monorepo Setup

```
monorepo/
├── .windsurf/rules/
│   ├── workspace.md (40 lines)
│   ├── frontend.md (70 lines)
│   └── backend.md (65 lines)
└── memory-bank/
    ├── INDEX.md
    ├── frontend/
    └── backend/
```

**workspace.md**:
```markdown
# Workspace Overview

[Minimal overview]

## Memory-Bank
See `memory-bank/INDEX.md`

[Core standards only]
```

**frontend.md**:
```markdown
# Frontend Rules

**Scope**: `frontend/**/*` files

[Minimal constraints + memory-bank references]
```

## Converting from Cursor

If you have Cursor rules:

```bash
# For each .mdc file
for file in .cursor/rules/*.mdc; do
    name=$(basename "$file" .mdc)
    
    # Remove YAML frontmatter (between --- markers)
    sed '/^---$/,/^---$/d' "$file" > ".windsurf/rules/${name}.md"
    
    # Add scope note manually to project-specific files
done
```

Then manually add scope notes to project files.

## Testing Your Configuration

1. **Open Windsurf** in your project
2. **Check loaded rules**: All `.windsurf/rules/*.md` files load
3. **Test memory-bank**: Ask "What documentation is available?"
4. **Verify behavior**: Ensure AI follows patterns and anti-patterns

## Common Issues

### Rules Not Loading

**Check**:
- Files are in `.windsurf/rules/` directory
- Files have `.md` extension (not `.md.template`)
- No frontmatter (Windsurf uses plain Markdown)
- No syntax errors

### Context Too Bloated

**Fix**:
- Reduce rule file sizes (< 75 lines each)
- Maximum 3-4 rule files
- Move details to `memory-bank/`

### Unclear Scope

**Fix**:
- Add scope notes to project-specific rules
- Make headers clear ("Frontend Rules", "Backend Rules")

## Next Steps

1. Fill in template placeholders
2. Create `memory-bank/INDEX.md` from template
3. Ensure total rules < 250 lines combined
4. Test configuration with real tasks
5. Iterate based on AI behavior
6. Add project-specific docs to `memory-bank/`

## References

- See `docs/tools/windsurf-guide.md` for comprehensive guide
- See `docs/fundamentals/memory-bank-pattern.md` for memory-bank details
- See `docs/best-practices/rule-writing.md` for writing effective rules
- See `examples/` for real-world configurations
