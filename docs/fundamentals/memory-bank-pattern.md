# The Memory-Bank Pattern

> **The most important pattern in context engineering**: External documentation organized for AI-optimized retrieval.

**Attribution**: This pattern emerged from production implementations in FanDuel nexus (Spark pipelines) and data-api (FastAPI platform) projects.

---

## What Is the Memory-Bank Pattern?

The memory-bank pattern separates **constraints** (in rules/CLAUDE.md) from **documentation** (in memory-bank/), enabling:

1. **Small context footprint**: Rules stay under 100 lines
2. **Just-in-time retrieval**: AI fetches docs when needed
3. **Progressive disclosure**: Navigate from overview to deep-dive
4. **Task-based navigation**: Find docs by intent, not structure

### The Problem It Solves

**Traditional Approach (Anti-Pattern)**:
```
.cursorrules (2,000 lines)
├── Project overview
├── Architecture details
├── All API endpoints
├── All transformations
├── Testing guide
├── Deployment guide
├── Troubleshooting guide
└── ... everything
```

**Issues**:
- Rules file bloats to thousands of lines
- Context window filled with mostly irrelevant info
- Updates require editing massive file
- Same info duplicated across tools (Cursor, Windsurf, Claude)

**Memory-Bank Approach**:
```
.cursor/rules/project.mdc (60 lines)
├── Core constraints
├── Anti-patterns
└── → "See memory-bank/INDEX.md for details"

memory-bank/
├── INDEX.md (< 100 lines) ← Navigation hub
├── architecture/
├── components/
└── workflows/
```

**Benefits**:
- Rules stay minimal (< 100 lines each)
- AI retrieves only relevant docs
- Single source of truth
- Easy to maintain and update

---

## Structure and Organization

### Canonical Structure

```
project/
├── .cursor/rules/          # Cursor constraints (< 100 lines each)
├── .windsurf/rules/        # Windsurf constraints (< 100 lines each)
├── CLAUDE.md               # Claude Code constraints + quick ref (< 300 lines)
└── memory-bank/
    ├── INDEX.md            # ⭐ Central navigation hub (< 100 lines)
    ├── architecture/       # System design docs
    │   ├── overview.md
    │   ├── data-flow.md
    │   └── decisions.md
    ├── components/         # Component-specific docs
    │   ├── component-a.md
    │   └── component-b.md
    ├── workflows/          # How-to guides
    │   ├── setup.md
    │   ├── testing.md
    │   └── deployment.md
    └── tech-stack/         # Technology guides
        ├── framework.md
        └── tools.md
```

### Directory Organization Principles

1. **By concern, not by size**: Split docs by topic, not length
2. **Flat when possible**: Avoid deep nesting (2-3 levels max)
3. **Descriptive names**: `adding-new-endpoint.md` not `guide-1.md`
4. **Consistent naming**: Use kebab-case for all files

---

## INDEX.md Design Principles

The INDEX.md is the **single most important file** in the memory-bank. It must be:

### 1. Navigation-First

**Bad INDEX.md** (structure-oriented):
```markdown
# Documentation

## Architecture
- overview.md
- data-flow.md

## Components
- kafka-reader.md
- delta-writer.md
```

**Good INDEX.md** (task-oriented):
```markdown
# Navigation by Task

| Task | Start With | Size |
|------|------------|------|
| Implementing Kafka reader | `components/kafka-reader.md` | 263 lines |
| Creating new transformation | `workflows/adding-transformation.md` | 365 lines |
| Debugging pipeline issues | `workflows/troubleshooting.md` | 468 lines |
```

**Why task-oriented wins**:
- Maps to **user intent** ("I want to X")
- Single entry point per task (no decision paralysis)
- Clear context budget (line counts)

### 2. Small and Scannable

- **Target**: < 100 lines
- **Maximum**: < 200 lines
- **Purpose**: Navigation only, not content

If your INDEX is getting large, you probably need project-specific sub-indexes.

### 3. Include File Sizes

Help AI budget context by categorizing docs:

```markdown
## File Size Reference

### Lightweight (< 150 lines) - Quick reads
| File | Lines | Topic |
|------|-------|-------|
| `architecture/overview.md` | 87 | System architecture |
| `components/auth.md` | 112 | Authentication |

### Medium (150-350 lines) - Focused deep-dives
| File | Lines | Topic |
|------|-------|-------|
| `workflows/adding-endpoint.md` | 365 | Endpoint creation |

### Large (350-500 lines) - Comprehensive references
| File | Lines | Topic |
|------|-------|-------|
| `workflows/troubleshooting.md` | 468 | Debugging guide |
```

### 4. Usage Strategy Section

Tell the AI **how** to use the memory-bank:

```markdown
## Usage Strategy

- Start here to locate documentation
- Use Read/Grep/Glob tools to fetch specific docs as needed
- Don't pre-load everything - retrieve based on current task
- Check file sizes to budget context appropriately
```

---

## Task-Based Navigation

### The Pattern

Map tasks to documentation, not structure to documentation.

**Example from data-api (production)**:

```markdown
## Navigation by Task

| Task | Start With | Size |
|------|------------|------|
| **Creating new API endpoint** | `workflows/adding-new-endpoint.md` | 365 lines |
| **Testing endpoints** | `workflows/testing-endpoints.md` | 360 lines |
| **Debugging/troubleshooting** | `workflows/troubleshooting.md` | 468 lines |
| **Code review** | `workflows/code-review-checklist.md` | 395 lines |
| **Understanding architecture** | `architecture/overview.md` | 87 lines |
| **Backend selection** | `architecture/data-backends.md` | 163 lines |
```

**Example from nexus (production)**:

```markdown
## Navigation by Task

| Task | Start With |
|------|------------|
| Implementing transformation | `fd-nexus/components/transformations_overview.md` |
| Creating new ingest script | `fd-nexus/scripts/ingest_patterns_overview.md` |
| Building Airflow DAG | `orbit-utils/workflow-patterns.md` |
| Understanding state pattern | `fd-nexus/features/state-pattern/` |
| Configuring environments | `orbit-utils/environment-handling.md` |
```

### Quick Start Paths

For complex tasks requiring multiple docs, provide step-by-step paths:

```markdown
## Quick Start Paths

### "I need to create a new API endpoint"
1. `workflows/adding-new-endpoint.md` - Step-by-step guide (365 lines)
2. `architecture/data-backends.md` - Choose backend (163 lines)
3. `tech-stack/jinja-templating.md` - SQL template syntax (460 lines)
4. `workflows/testing-endpoints.md` - Testing strategy (360 lines)

### "I need to debug a failing endpoint"
1. `workflows/troubleshooting.md` - Common issues and solutions (468 lines)
2. `tech-stack/observability.md` - Datadog APM traces (462 lines)
3. Component doc for specific facade
```

---

## File Size Categorization

### The 4-Level Hierarchy

Based on nexus production implementation:

**Level 0: Index Files** (< 60 lines)
- **Purpose**: Navigation only
- **Example**: `memory-bank/INDEX.md`
- **When to read**: Always start here

**Level 1: Quick References** (< 150 lines)
- **Purpose**: Copy-paste solutions for common tasks
- **Example**: `quick-refs/common_transformations.md`
- **When to read**: Know what to do, need syntax/example

**Level 2: Component Overviews** (150-350 lines)
- **Purpose**: Understanding framework architecture
- **Example**: `components/transformations_overview.md`
- **When to read**: Need to understand how something works

**Level 3: Deep References** (350+ lines)
- **Purpose**: Comprehensive specifications
- **Example**: `features/state-pattern/implementation.md` (1,400 lines)
- **When to read**: Complex implementation or troubleshooting

### Categorization in INDEX.md

```markdown
## File Size Reference

### Lightweight (< 150 lines) - Quick reads
- Fast to load, minimal context cost
- Use for: quick lookups, syntax references
- Files: [list with line counts]

### Medium (150-350 lines) - Focused deep-dives
- Moderate context cost
- Use for: understanding components, workflows
- Files: [list with line counts]

### Large (350-500 lines) - Comprehensive references
- High context cost
- Use for: complex workflows, troubleshooting
- Files: [list with line counts]

### Very Large (> 500 lines) - Detailed specifications
- Very high context cost
- Use for: implementation details, complete specs
- Files: [list with line counts]
```

---

## Progressive Disclosure Strategy

### The Principle

AI should discover information in layers, from general to specific.

### Implementation

**1. Start with INDEX.md** (< 100 lines)
- Scans available documentation
- Identifies relevant docs for current task
- Checks file sizes to budget context

**2. Read relevant overview** (150-350 lines)
- Understands component/workflow architecture
- Identifies specific sections to dive into
- Avoids reading irrelevant details

**3. Target specific sections** (as needed)
- Uses grep/search to find exact location
- Reads only relevant portions
- Expands if needed

### Example Flow

Task: "Implement retry logic for Kafka reader"

**Traditional approach** (anti-pattern):
```
1. Load all Kafka reader documentation (2,000 lines)
2. Load all error handling patterns (1,500 lines)
3. Load all retry examples (800 lines)
→ 4,300 lines in context, mostly irrelevant
```

**Memory-bank approach**:
```
1. Read INDEX.md (56 lines)
   → "For Kafka readers, see components/kafka-readers.md"

2. Read components/kafka-readers.md (263 lines)
   → "Retry logic in JdbcReader at line 45-67"

3. Read JdbcReader lines 45-67 (23 lines)
   → Understand pattern

4. Implement following pattern
→ 342 lines total, all highly relevant
```

---

## Real Examples from Production

### Example 1: nexus (Monorepo, Spark Pipelines)

**Workspace INDEX** (`memory-bank/INDEX.md` - 56 lines):

```markdown
# Workspace Memory-Bank Index

**Purpose:** Central navigation for context retrieval across all projects

**Usage Strategy:**
- Start here to locate project-specific documentation
- Use Read/Grep/Glob tools to fetch specific documentation as needed
- Don't pre-load everything - retrieve based on current task

---

## Project Documentation

### fd-nexus (Spark Library)
**Index:** `fd-nexus/INDEX.md`
**Focus:** Data pipeline components, transformations, readers/writers

### orbit-utils (Airflow Operators)
**Index:** `orbit-utils/INDEX.md`
**Focus:** Databricks operators, workflow patterns

---

## Navigation by Task

| Task | Start With |
|------|------------|
| Implementing transformation | `fd-nexus/components/transformations_overview.md` |
| Creating new ingest script | `fd-nexus/scripts/ingest_patterns_overview.md` |
| Building Airflow DAG | `orbit-utils/workflow-patterns.md` |
```

**Project INDEX** (`memory-bank/fd-nexus/INDEX.md` - 166 lines):
- Task-based navigation for fd-nexus specifics
- File size categories
- Quick reference section

**Why this works**:
- Workspace INDEX: 56 lines (< 1 minute to read)
- Routes to project-specific INDEX
- Each project has its own navigation

### Example 2: data-api (FastAPI Platform)

**Single INDEX** (`memory-bank/INDEX.md` - 157 lines):

```markdown
# Data API Platform Memory-Bank Index

**Purpose**: Central navigation hub for context retrieval

**Usage Strategy**:
- Start here to locate task-specific documentation
- Use Read/Grep/Glob tools to fetch specific docs
- Check file sizes to budget context

---

## Navigation by Task

| Task | Start With | Size |
|------|------------|------|
| **Creating new endpoint** | `workflows/adding-new-endpoint.md` | 365 lines |
| **Testing** | `workflows/testing-endpoints.md` | 360 lines |
| **Debugging** | `workflows/troubleshooting.md` | 468 lines |

---

## File Size Reference

### Lightweight (< 150 lines) - Quick reads
| File | Lines | Topic |
|------|-------|-------|
| `architecture/overview.md` | 87 | System architecture |

### Medium (150-350 lines) - Focused deep-dives
| File | Lines | Topic |
|------|-------|-------|
| `workflows/adding-new-endpoint.md` | 365 | Endpoint creation |

---

## Quick Start Paths

### "I need to create a new API endpoint"
1. `workflows/adding-new-endpoint.md` (365 lines)
2. `architecture/data-backends.md` (163 lines)
3. `tech-stack/jinja-templating.md` (460 lines)
```

**Why this works**:
- Single project: single INDEX
- Task table includes line counts
- Quick Start Paths for common flows
- Total: 21 files, ~6,049 lines

---

## Implementation Guide

### Step 1: Create Directory Structure

```bash
mkdir -p memory-bank/{architecture,components,workflows,tech-stack}
```

### Step 2: Create INDEX.md Template

```markdown
# [Project] Memory-Bank Index

**Purpose**: Central navigation for context retrieval

**Usage Strategy**:
- Start here to locate documentation
- Use Read/Grep/Glob tools to fetch specific docs as needed
- Don't pre-load everything - retrieve based on current task

---

## Navigation by Task

| Task | Start With | Size |
|------|------------|------|
| [Task 1] | `path/to/doc.md` | [N lines] |
| [Task 2] | `path/to/doc.md` | [N lines] |

---

## File Size Reference

### Lightweight (< 150 lines) - Quick reads
- [file1.md] ([N lines]) - [topic]

### Medium (150-350 lines) - Focused deep-dives
- [file1.md] ([N lines]) - [topic]

### Large (350-500 lines) - Comprehensive references
- [file1.md] ([N lines]) - [topic]

---

## Directory Structure

```
memory-bank/
├── architecture/
├── components/
├── workflows/
└── tech-stack/
```
```

### Step 3: Create Documentation Files

Organize by concern:
- **architecture/**: System design, decisions, data flow
- **components/**: Component-specific documentation
- **workflows/**: How-to guides, procedures
- **tech-stack/**: Technology-specific guides

### Step 4: Update Rules to Reference Memory-Bank

**Cursor** (`.cursor/rules/project.mdc`):
```yaml
---
description: Project rules
globs: ["**/*"]
---

# Project Rules

## Memory-Bank Navigation
**Start with** `memory-bank/INDEX.md` for detailed documentation.

## Core Constraints
[Your constraints here - keep under 100 lines total]
```

**Claude Code** (`CLAUDE.md`):
```markdown
# CLAUDE.md

## Memory-Bank Navigation
Start with `memory-bank/INDEX.md` for detailed documentation.

## Quick Reference
[Common commands, patterns - keep under 300 lines total]
```

### Step 5: Maintain and Update

- **Add line counts** when creating new docs
- **Update INDEX.md** when adding new docs
- **Keep INDEX.md small** (< 100 lines ideally)
- **Split large docs** into focused pieces

---

## Common Pitfalls

### 1. Making INDEX Too Large

```
❌ INDEX.md with 500 lines
✅ INDEX.md with < 100 lines
```

If your INDEX is large, you need sub-indexes or better organization.

### 2. Embedding Content in INDEX

```
❌ INDEX.md contains actual documentation
✅ INDEX.md contains only navigation pointers
```

INDEX should be a map, not the territory.

### 3. Structure-Based Instead of Task-Based

```
❌ Navigation by directory structure
✅ Navigation by user intent/task
```

Users think in tasks ("I want to X"), not structure ("I wonder what's in /components").

### 4. Missing File Sizes

```
❌ Just list files
✅ Include line counts for context budgeting
```

File sizes help AI decide what to read based on available context.

### 5. No Usage Strategy

```
❌ Just a list of files
✅ Explicit instructions on how to use the memory-bank
```

Tell the AI how to navigate and retrieve information.

---

## Benefits Summary

| Benefit | Impact |
|---------|--------|
| **Small rules** | < 100 lines each, no context bloat |
| **Just-in-time retrieval** | Load only what's needed |
| **Single source of truth** | Same docs for all tools |
| **Easy maintenance** | Update docs, not rules |
| **Better AI performance** | High signal-to-noise ratio |
| **Context budgeting** | File sizes help plan retrieval |
| **Progressive disclosure** | Navigate from overview to detail |

---

## References

- See `docs/fundamentals/context-engineering.md` for underlying principles
- See `templates/memory-bank/` for ready-to-use templates
- See `examples/` for real-world implementations

**Production Implementations**:
- `examples/monorepo-data-platform/` - Based on FanDuel nexus
- `examples/fastapi-microservice/` - Based on data-api platform
