# Memory-Bank Index

**Purpose**: Central navigation for project documentation

**Usage Strategy**:
- Start here to locate documentation
- Use Read tool to fetch specific docs as needed
- Don't pre-load everything - retrieve based on current task

---

## Navigation by Task

| Task | Start With | Size |
|------|------------|------|
| **Understanding architecture** | `architecture.md` | 120 lines |
| **Adding new API endpoint** | `api-patterns.md` | 185 lines |
| **Writing tests** | `testing.md` | 95 lines |
| **Deploying** | `deployment.md` | 110 lines |

---

## File Size Reference

### Lightweight (< 150 lines) - Quick reads
**Token cost**: ~500 tokens each  
**When to use**: Quick lookups, how-to guides

- `architecture.md` (120 lines) - System architecture and design
- `testing.md` (95 lines) - Testing patterns and setup
- `deployment.md` (110 lines) - Deployment process

### Medium (150-350 lines) - Focused deep-dives
**Token cost**: ~1,000 tokens  
**When to use**: Implementing features, understanding patterns

- `api-patterns.md` (185 lines) - API endpoint patterns and examples

---

## Files

```
memory-bank/
├── INDEX.md           ← You are here
├── architecture.md    ← Hexagonal design, request flow
├── api-patterns.md    ← How to add endpoints, patterns
├── testing.md         ← pytest setup, test patterns
└── deployment.md      ← Docker, environment, deployment
```

**Total**: 5 files, ~575 lines

---

## Quick Start Paths

### "I need to add a new API endpoint"
1. `api-patterns.md` - Endpoint creation pattern (185 lines)
2. `testing.md` - How to test it (95 lines)

### "I need to understand the architecture"
1. `architecture.md` - System design (120 lines)
2. `api-patterns.md` - API layer details (185 lines)

### "I need to deploy changes"
1. `deployment.md` - Deployment process (110 lines)

---

## See Also

- **Rules**: `.cursor/rules/project.mdc`, `.windsurf/rules/project.md`, `CLAUDE.md`
- **README**: Project overview and setup
