# Simple Project Example

Minimal context engineering setup for a single-project repository.

---

## Project Overview

**Type**: Single Python project (FastAPI REST API)  
**Size**: Small (~5,000 lines of code)  
**Team**: 2-3 developers  
**Tools**: Cursor, Windsurf, Claude Code (all three supported)

---

## What's Included

This example demonstrates:
- ✅ Minimal rule configuration (< 100 lines total)
- ✅ Simple memory-bank structure (5 files)
- ✅ Tool-agnostic approach (same content, different formats)
- ✅ Task-based navigation
- ✅ Quick setup (< 15 minutes)

---

## Directory Structure

```
simple-project/
├── .cursor/rules/
│   └── project.mdc (75 lines)
├── .windsurf/rules/
│   └── project.md (70 lines)
├── CLAUDE.md (145 lines)
└── memory-bank/
    ├── INDEX.md (65 lines)
    ├── architecture.md (120 lines)
    ├── api-patterns.md (185 lines)
    ├── testing.md (95 lines)
    └── deployment.md (110 lines)
```

**Total**:
- Rules: 75 lines (Cursor), 70 lines (Windsurf), 145 lines (Claude)
- Memory-bank: 5 files, 575 lines

---

## Key Features

### 1. Single Rule File per Tool

**Why**: Small project doesn't need multiple rule files.

**Cursor** (`.cursor/rules/project.mdc`):
- Uses `alwaysApply: true` (not globs)
- Project overview, patterns, anti-patterns
- 75 lines total

**Windsurf** (`.windsurf/rules/project.md`):
- Same content as Cursor (no YAML)
- 70 lines total

**Claude Code** (`CLAUDE.md`):
- Includes development commands
- 145 lines total

### 2. Flat Memory-Bank Structure

No subdirectories needed for small project:

```
memory-bank/
├── INDEX.md              ← Navigation
├── architecture.md       ← System design
├── api-patterns.md       ← How to add endpoints
├── testing.md            ← How to test
└── deployment.md         ← How to deploy
```

### 3. Task-Based INDEX

```markdown
## Navigation by Task

| Task | Start With | Size |
|------|------------|------|
| Understanding architecture | `architecture.md` | 120 lines |
| Adding new endpoint | `api-patterns.md` | 185 lines |
| Writing tests | `testing.md` | 95 lines |
| Deploying | `deployment.md` | 110 lines |
```

---

## Files Explained

### .cursor/rules/project.mdc

**Sections**:
1. Project Context (tech stack, architecture)
2. Key Patterns (3 most important)
3. Memory-Bank Reference
4. Development Commands
5. Anti-Patterns (3 most dangerous)

**Size**: 75 lines

**YAML frontmatter**:
```yaml
---
description: Simple FastAPI project rules
alwaysApply: true
---
```

### .windsurf/rules/project.md

**Same content as Cursor** (95% identical):
- No YAML frontmatter
- Otherwise identical

**Size**: 70 lines

### CLAUDE.md

**Additional content**:
- Development commands with examples
- Quick reference section
- Critical constraints (security, performance)

**Size**: 145 lines

### memory-bank/INDEX.md

**Sections**:
1. Usage Strategy
2. Navigation by Task
3. File Size Reference
4. Directory Structure (flat, just list files)

**Size**: 65 lines

### memory-bank/*.md

**architecture.md** (120 lines):
- FastAPI + PostgreSQL architecture
- Request flow
- Component responsibilities

**api-patterns.md** (185 lines):
- How to add new endpoints
- Request/response patterns
- Error handling
- Authentication

**testing.md** (95 lines):
- pytest setup
- Test patterns (unit, integration)
- Running tests

**deployment.md** (110 lines):
- Docker setup
- Environment variables
- Deployment process

---

## Quick Start

### 1. Copy Files

```bash
# Copy rules for your tool
cp examples/simple-project/.cursor/rules/project.mdc .cursor/rules/
# OR
cp examples/simple-project/.windsurf/rules/project.md .windsurf/rules/
# OR
cp examples/simple-project/CLAUDE.md .
```

### 2. Copy Memory-Bank

```bash
cp -r examples/simple-project/memory-bank/ ./
```

### 3. Customize

**Edit rule file**:
- Replace tech stack with yours
- Update patterns for your project
- Customize anti-patterns

**Edit memory-bank files**:
- Update architecture.md with your design
- Update api-patterns.md with your conventions
- Update testing.md with your approach
- Update deployment.md with your process

### 4. Test

```
Open your AI tool
Ask: "What documentation is available for adding a new endpoint?"
Expected: References memory-bank/INDEX.md → api-patterns.md
```

---

## Adaptation Guide

### For Different Tech Stacks

**Replace** in rules:
- Python/FastAPI → Your language/framework
- pytest → Your test framework
- PostgreSQL → Your database

**Update** in memory-bank:
- architecture.md → Your architecture
- api-patterns.md → Your component patterns
- testing.md → Your testing approach

### For Larger Projects

**When to split into multiple files**:
- Project grows to 10,000+ lines
- Multiple distinct components
- Monorepo with subprojects

**Then**:
- Create workspace rule (always apply)
- Create component rules (glob-based for Cursor)
- Organize memory-bank by component

---

## Success Metrics

After setup, you should see:

- ✅ First-attempt success > 70%
- ✅ Context utilization < 15% before starting work
- ✅ AI references memory-bank appropriately
- ✅ Setup time < 15 minutes
- ✅ Team productive immediately

---

## Common Questions

### Q: Do I need all three tool configs?

**A**: No, just the one(s) your team uses. But adding others takes only 5 minutes.

### Q: Can I skip the memory-bank?

**A**: Not recommended. Even small projects benefit from separating rules (constraints) from documentation (how things work).

### Q: My project is even smaller. Can I simplify further?

**A**: Yes! Minimal viable setup:
- 1 rule file (< 50 lines)
- memory-bank/INDEX.md with pointers to README

### Q: How do I maintain this?

**A**: 
- Monthly: Review rules, update for new patterns
- Quarterly: Review memory-bank, update docs
- As needed: Add rules when AI makes repeated mistakes

---

## Next Steps

1. Copy files to your project
2. Customize for your tech stack
3. Test with real tasks
4. Iterate based on AI behavior
5. Share learnings!

---

## References

- See `../../docs/` for detailed guides
- See `../../templates/` for blank templates
- See other examples for larger projects
