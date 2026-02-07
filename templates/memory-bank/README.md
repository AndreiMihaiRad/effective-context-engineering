# Memory-Bank Template

Ready-to-use template for the memory-bank pattern - the foundation of effective context engineering.

## File Included

- `INDEX.md.template` - Central navigation hub

## What Is the Memory-Bank Pattern?

**The memory-bank pattern separates**:
- **Constraints** (in rules/CLAUDE.md) - What to do/not do
- **Documentation** (in memory-bank/) - How things work

**Benefits**:
- Rules stay small (< 100 lines)
- AI retrieves docs just-in-time
- Single source of truth
- Tool-agnostic (same for Cursor/Windsurf/Claude)

## Quick Start

### 1. Create Memory-Bank Directory

```bash
mkdir -p memory-bank/{architecture,components,workflows,tech-stack}
```

### 2. Copy Template

```bash
cp path/to/templates/memory-bank/INDEX.md.template memory-bank/INDEX.md
```

### 3. Fill in Placeholders

Replace all `[PLACEHOLDER]` values in `INDEX.md`:

**Navigation by Task section**:
- `[Task 1]`, `[Task 2]`, etc. - Common tasks users perform
- `[path/to/doc.md]` - Path to documentation for that task
- `[N lines]` - Line count of that doc (for context budgeting)

**File Size Reference sections**:
- `[category]/[file].md` - Your actual docs
- `[N]` - Line count
- `[Topic description]` - What the doc covers

**Directory Structure**:
- Update tree view to match your structure
- Add descriptions for each directory

**Quick Start Paths**:
- Define step-by-step paths for common workflows
- Include file paths and line counts

### 4. Create Documentation Files

```bash
# Example structure
touch memory-bank/architecture/overview.md
touch memory-bank/components/component-a.md
touch memory-bank/workflows/workflow-1.md
```

### 5. Add Line Counts

```bash
# Count lines in each doc
wc -l memory-bank/**/*.md

# Update INDEX.md with actual line counts
```

## Template Details

### INDEX.md.template

**Purpose**: Central navigation hub for all documentation

**Key sections**:
1. **Usage Strategy** - How to use the memory-bank
2. **Navigation by Task** - Map tasks to docs
3. **File Size Reference** - Categorized by size with line counts
4. **Directory Structure** - Tree view of organization
5. **Quick Start Paths** - Step-by-step for common tasks
6. **Cross-References** - Related docs
7. **See Also** - Links to rules, README

**Target size**: < 100 lines (can go up to 150 for complex projects)

## Directory Organization

### Recommended Structure

```
memory-bank/
├── INDEX.md              ← Navigation hub
├── architecture/         ← System design docs
│   ├── overview.md
│   ├── decisions.md
│   └── data-flow.md
├── components/           ← Component-specific docs
│   ├── component-a.md
│   └── component-b.md
├── workflows/            ← How-to guides
│   ├── setup.md
│   ├── testing.md
│   └── deployment.md
└── tech-stack/           ← Technology guides
    ├── framework.md
    └── tools.md
```

### Directory Principles

1. **By concern, not size** - Organize by topic
2. **Flat when possible** - Avoid deep nesting (2-3 levels max)
3. **Descriptive names** - Use kebab-case, be clear
4. **Consistent structure** - Same across projects

## File Size Categories

### The 4-Tier System

**Lightweight (< 150 lines)**:
- Token cost: ~500 tokens
- Purpose: Quick lookups, syntax
- Examples: Overview, quick reference

**Medium (150-350 lines)**:
- Token cost: ~1,000-1,500 tokens
- Purpose: Understanding components
- Examples: Component guides, workflows

**Large (350-500 lines)**:
- Token cost: ~1,500-2,000 tokens
- Purpose: Comprehensive references
- Examples: Troubleshooting, complex workflows

**Very Large (> 500 lines)**:
- Token cost: 2,000+ tokens
- Purpose: Detailed specifications
- Examples: Complete API reference, detailed specs
- Caution: Only read when necessary

## Task-Based Navigation

### The Pattern

Map **user intent** to **documentation**, not directory structure.

**Bad** (structure-oriented):
```markdown
## Documentation

### Architecture
- overview.md
- data-flow.md

### Components
- kafka.md
- redis.md
```

**Good** (task-oriented):
```markdown
## Navigation by Task

| Task | Start With | Size |
|------|------------|------|
| Implementing Kafka reader | `components/kafka.md` | 263 lines |
| Understanding data flow | `architecture/data-flow.md` | 145 lines |
| Debugging issues | `workflows/troubleshooting.md` | 468 lines |
```

**Why**: Users think "I want to X", not "I wonder what's in /components".

## Real-World Examples

### Example 1: Small Project

**Structure**:
```
memory-bank/
├── INDEX.md (80 lines)
├── architecture.md (120 lines)
├── testing.md (95 lines)
└── deployment.md (110 lines)
```

**INDEX.md**:
```markdown
## Navigation by Task

| Task | Start With | Size |
|------|------------|------|
| Understanding architecture | `architecture.md` | 120 lines |
| Writing tests | `testing.md` | 95 lines |
| Deploying | `deployment.md` | 110 lines |
```

### Example 2: Medium Project (data-api)

**Structure**:
```
memory-bank/
├── INDEX.md (157 lines)
├── architecture/ (4 files, ~461 lines)
├── components/ (10 files, ~2,553 lines)
├── workflows/ (4 files, ~1,588 lines)
└── tech-stack/ (3 files, ~1,447 lines)
```

**Total**: 21 files, ~6,049 lines

**INDEX.md** features:
- Task-based navigation with line counts
- File size categories
- Quick Start Paths for common workflows

### Example 3: Large Monorepo (nexus)

**Structure**:
```
memory-bank/
├── INDEX.md (56 lines, workspace level)
├── fd-nexus/
│   ├── INDEX.md (166 lines, project level)
│   ├── components/ (10+ files)
│   ├── features/ (5+ directories)
│   └── quick-refs/ (< 100 lines each)
└── orbit-utils/
    ├── INDEX.md
    └── [docs]
```

**Two-level INDEX**:
- Workspace INDEX → Routes to project INDEXes
- Project INDEXes → Routes to specific docs

## Integration with Rules

### Cursor (.cursor/rules/project.mdc)

```yaml
---
description: Project rules
globs: ["project/**/*"]
---

# Project Rules

## Memory-Bank First
**ALWAYS consult** `memory-bank/project/INDEX.md` before implementation

## Core Constraints
[< 50 lines of constraints]
```

### Windsurf (.windsurf/rules/project.md)

```markdown
# Project Rules

## Memory-Bank First
For detailed documentation, see `memory-bank/project/INDEX.md`

## Core Constraints
[< 30 lines of constraints]
```

### Claude Code (CLAUDE.md)

```markdown
# CLAUDE.md

## Memory-Bank Navigation
Start with `memory-bank/INDEX.md` for detailed documentation.

## Quick Reference
[< 50 lines of common tasks]
```

## Best Practices

### 1. Keep INDEX Under 100 Lines

```
Target: < 100 lines
Maximum: < 150 lines (complex projects)

If larger:
- Too much content (move to docs)
- Too many files (reorganize)
```

### 2. Include All Line Counts

```markdown
| File | Lines | Topic |
|------|-------|-------|
| `overview.md` | 87 | System architecture |  ← Include line count!
```

Helps AI budget context.

### 3. Task-Based, Not Structure-Based

Map to user intent, not directory layout.

### 4. Add Usage Strategy

```markdown
## Usage Strategy
- Start here to locate documentation
- Use Read/Grep/Glob tools to fetch specific docs
- Don't pre-load everything
- Check file sizes to budget context
```

### 5. Maintain Cross-References

```markdown
## Cross-References

| If you're reading... | Also consider... |
|---------------------|------------------|
| `adding-endpoint.md` | `testing.md`, `deployment.md` |
```

## Testing Your Memory-Bank

```
# Test 1: Can AI find docs?
"What documentation is available for adding an endpoint?"

Expected: References INDEX.md, finds adding-endpoint.md

# Test 2: Does AI understand file sizes?
"Give me a quick overview of the architecture"

Expected: Reads lightweight overview.md, not comprehensive 500-line spec

# Test 3: Does AI follow navigation?
"I need to debug a failing endpoint"

Expected: Follows task table → troubleshooting.md
```

## Maintenance

### When to Update

- **After adding docs**: Update INDEX with new file
- **After major changes**: Update line counts
- **Monthly review**: Verify structure still makes sense
- **Quarterly**: Comprehensive review and cleanup

### Keeping Line Counts Accurate

```bash
# Update all line counts
for file in memory-bank/**/*.md; do
    lines=$(wc -l < "$file")
    echo "$file: $lines lines"
done

# Update INDEX.md with actual counts
```

## Common Issues

### INDEX Too Large

**Fix**:
- Remove content (INDEX is navigation only)
- Split into project-specific INDEXes (for monorepos)

### AI Doesn't Use Memory-Bank

**Fix**:
- Add explicit rule: "ALWAYS consult memory-bank/INDEX.md"
- Test with specific question

### Unclear Task Mapping

**Fix**:
- Make task names more specific
- Add more tasks to navigation table
- Include common variations

## Next Steps

1. Copy `INDEX.md.template` to `memory-bank/INDEX.md`
2. Create directory structure
3. Write your first docs
4. Count lines with `wc -l`
5. Update INDEX.md with actual paths and counts
6. Reference from rules/CLAUDE.md
7. Test with real tasks
8. Iterate based on usage

## References

- See `docs/fundamentals/memory-bank-pattern.md` for comprehensive guide
- See `docs/best-practices/context-budgeting.md` for file sizing guidance
- See `examples/` for real-world memory-bank implementations
