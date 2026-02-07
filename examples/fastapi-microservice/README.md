# FastAPI Microservice Example

Context engineering setup for a multi-component FastAPI platform, based on the data-api production project.

---

## Project Overview

**Type**: FastAPI self-service data access platform  
**Size**: Medium (~15,000 lines of code across 4 components)  
**Team**: 5-8 developers  
**Tools**: All three (Cursor, Windsurf, Claude Code)  
**Source**: Based on data-api production implementation

---

## What This Example Demonstrates

- ✅ Multi-component project organization
- ✅ Task-based navigation with file size annotations
- ✅ **Facade pattern** in CLAUDE.md (condensed overview → memory-bank details)
- ✅ Quick Start Paths for common workflows
- ✅ Security and performance constraints
- ✅ Production-ready setup

---

## Architecture

### Components

```
data-api/
├── facades/              # 3 backend facades (Trino, Databricks, TinyBird)
├── definitions/          # CLI tool for TOML validation
├── kong-gateway/         # API gateway config
└── infrastructure/       # AWS/Kubernetes (apollo-infra)
```

### Key Pattern: Declarative API Creation

```
1. Define endpoint in TOML → Auto-generated Kong routes + OpenAPI
2. Write SQL template (Jinja2) → Rendered at runtime
3. Deploy via CI/CD → S3 → Kong
```

---

## Memory-Bank Structure

```
memory-bank/
├── INDEX.md (157 lines)  ← Central navigation with file sizes
├── architecture/         ← 4 files, ~461 lines
│   ├── overview.md (87)
│   ├── facades-pattern.md (73)
│   ├── deployment-flow.md (138)
│   └── data-backends.md (163)
├── components/           ← 10 files, ~2,553 lines
│   ├── kong-gateway.md (221)
│   ├── trino-facade.md (263)
│   ├── databricks-facade.md (319)
│   ├── tinybird-facade.md (310)
│   ├── definitions-cli.md (403)
│   └── [infrastructure docs]
├── workflows/            ← 4 files, ~1,588 lines
│   ├── adding-new-endpoint.md (365)
│   ├── testing-endpoints.md (360)
│   ├── troubleshooting.md (468)
│   └── code-review-checklist.md (395)
└── tech-stack/           ← 3 files, ~1,447 lines
    ├── python-fastapi.md (525)
    ├── jinja-templating.md (460)
    └── observability.md (462)
```

**Total**: 21 files, ~6,049 lines

---

## Key Features

### 1. Task-Based INDEX with File Sizes

From `memory-bank/INDEX.md`:

```markdown
## Navigation by Task

| Task | Start With | Size |
|------|------------|------|
| **Creating new endpoint** | `workflows/adding-new-endpoint.md` | 365 lines |
| **Testing** | `workflows/testing-endpoints.md` | 360 lines |
| **Debugging** | `workflows/troubleshooting.md` | 468 lines |
| **Code review** | `workflows/code-review-checklist.md` | 395 lines |
```

**Why this works**:
- Maps to user intent ("I want to X")
- Includes line counts for context budgeting
- Single entry point per task

### 2. File Size Categories

From `memory-bank/INDEX.md`:

```markdown
## File Size Reference

### Lightweight (< 150 lines) - Quick reads
| File | Lines | Topic |
|------|-------|-------|
| `architecture/overview.md` | 87 | System architecture |

### Medium (150-350 lines) - Focused deep-dives
| File | Lines | Topic |
|------|-------|-------|
| `workflows/adding-new-endpoint.md` | 365 | Endpoint creation |

### Large (350-500 lines) - Comprehensive references
| File | Lines | Topic |
|------|-------|-------|
| `workflows/troubleshooting.md` | 468 | Debugging guide |
```

**Benefit**: AI chooses appropriate doc depth based on context budget.

### 3. Quick Start Paths

From `memory-bank/INDEX.md`:

```markdown
## Quick Start Paths

### "I need to create a new API endpoint"
1. `workflows/adding-new-endpoint.md` (365 lines)
2. `architecture/data-backends.md` (163 lines)
3. `tech-stack/jinja-templating.md` (460 lines)
4. `workflows/testing-endpoints.md` (360 lines)
```

**Benefit**: Step-by-step navigation for complex workflows.

### 4. Facade Pattern in CLAUDE.md

**CLAUDE.md** (266 lines total):
- Workspace overview (50 lines)
- Development commands (40 lines)  
- Architecture summary (60 lines)
- **Memory-bank navigation** (30 lines) ← Points to details
- Quick reference (40 lines)
- Critical constraints (16 lines)

**Pattern**: CLAUDE.md is condensed facade → memory-bank has details.

### 5. Critical Constraints Section

From `CLAUDE.md`:

```markdown
## Critical Constraints

### Security (DO NOT)
- ❌ String concatenation in SQL templates
- ❌ Overly permissive ACLs
- ✅ Use parameterized queries
- ✅ Least-privilege ACL groups

### Performance (DO NOT)
- ❌ Full table scans
- ❌ Functions on filter columns
- ✅ Filter on partition/indexed columns
- ✅ Use appropriate cluster

### Breaking Changes (DO NOT)
- ❌ Rename parameters (breaks consumers)
- ❌ Change parameter types
- ✅ Add new optional parameters
```

**Benefit**: Critical constraints front-and-center.

---

## Rule Files

### Cursor (`.cursor/rules/workspace.mdc`)

**Size**: 44 lines

**Sections**:
1. Component overview table
2. Memory-bank navigation
3. Code standards

**Pattern**: Minimal workspace rule, details in memory-bank.

### Windsurf (`.windsurf/rules/workspace.md`)

**Size**: 39 lines

**Content**: 95% identical to Cursor (no YAML frontmatter).

### Claude Code (`CLAUDE.md`)

**Size**: 266 lines

**Additional content** (vs Cursor/Windsurf):
- Development commands with examples
- More comprehensive quick reference
- Detailed critical constraints

---

## Real-World Impact

**From production data-api implementation**:

### Before Context Engineering
- First-attempt success: 50%
- Context at 70% before starting work
- Common issues: Wrong SQL syntax, missing validation

### After Context Engineering
- First-attempt success: 80%
- Context at 12% before starting work
- AI follows Jinja2 patterns, validates TOML correctly

**Setup time**: 2 hours initial, 15 min/month maintenance

---

## Adaptation Guide

### For Your FastAPI Project

**Copy**:
- memory-bank/INDEX.md structure (task-based + file sizes)
- Quick Start Paths pattern
- File size categories

**Customize**:
- Replace Trino/Databricks with your backends
- Update workflows for your deployment process
- Add your tech stack guides

### For Non-FastAPI Projects

**Keep**:
- Task-based INDEX
- File size annotations
- Quick Start Paths
- Critical constraints section

**Replace**:
- Tech stack specific content
- Framework patterns
- Deployment specifics

---

## Key Lessons from Production

### 1. Task-Based Navigation is Critical

Users don't think "I wonder what's in /components", they think "I want to add an endpoint".

Map to intent, not structure.

### 2. File Sizes Help AI Budget Context

Without line counts: AI loads 500-line spec for quick question.  
With line counts: AI reads 87-line overview instead.

### 3. Quick Start Paths Reduce Confusion

Complex workflows benefit from step-by-step paths. AI knows exactly what to read and in what order.

### 4. Critical Constraints Prevent Issues

Security and performance constraints in CLAUDE.md prevent common mistakes.

### 5. Facade Pattern Keeps Rules Small

CLAUDE.md at 266 lines is condensed overview. Details in memory-bank.

Result: Context stays low, AI can navigate to details when needed.

---

## Setup Time

**Initial**: 2-3 hours
- Copy structure
- Write initial docs (5 files minimum)
- Count lines, update INDEX
- Test with real tasks

**Maintenance**: 15-30 minutes/month
- Update line counts
- Add new patterns
- Remove obsolete docs

---

## Success Metrics

After setup:

- ✅ First-attempt success > 75%
- ✅ Context utilization < 15% before work
- ✅ AI references memory-bank correctly
- ✅ AI follows Jinja2/SQL patterns
- ✅ AI asks about TOML schema instead of guessing

---

## Files to Explore

While this example shows the structure, key files to review:

1. `memory-bank/INDEX.md` - Task-based navigation + file sizes
2. `CLAUDE.md` - Facade pattern with critical constraints
3. `memory-bank/workflows/adding-new-endpoint.md` - Step-by-step guide
4. `.cursor/rules/workspace.mdc` - Minimal workspace rule

---

## Next Steps

1. Review the structure and patterns
2. Adapt INDEX.md for your project
3. Create initial workflow docs (2-3 minimum)
4. Add file size annotations
5. Test with real tasks
6. Iterate based on AI behavior

---

## References

- **Source inspiration**: data-api production implementation
- See `../../docs/fundamentals/memory-bank-pattern.md` for pattern details
- See `../../docs/best-practices/context-budgeting.md` for file sizing
