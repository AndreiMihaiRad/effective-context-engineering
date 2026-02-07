# Monorepo Data Platform Example

Context engineering setup for a large monorepo with multiple projects, based on the FanDuel nexus production implementation.

---

## Project Overview

**Type**: Enterprise data platform monorepo  
**Size**: Large (~50,000 lines of code across 2 projects)  
**Team**: 10-15 developers  
**Tools**: All three (Cursor, Windsurf, Claude Code)  
**Source**: Based on FanDuel nexus production implementation

---

## What This Example Demonstrates

- ✅ **Monorepo organization** with workspace + project rules
- ✅ **Hierarchical memory-bank** (workspace INDEX → project INDEXes)
- ✅ **4-level documentation hierarchy** (index → quick refs → overviews → deep refs)
- ✅ **Glob-based conditional loading** (Cursor)
- ✅ **50+ transformation registry** pattern
- ✅ Production-scale setup

---

## Architecture

### Monorepo Structure

```
nexus/
├── fd-nexus/          # Spark library for Databricks pipelines
│   └── src/
│       ├── readers/   # Kafka, JDBC, S3 readers
│       ├── writers/   # Delta Lake writers
│       ├── transforms/ # 50+ transformations
│       └── state/     # Distributed state management
├── orbit-utils/       # Airflow operators
│   └── operators/
│       ├── databricks/  # Databricks job operators
│       ├── dbt/         # dbt operators
│       └── hightouch/   # Reverse ETL operators
└── shared/            # Shared utilities
```

### Integration

**orbit-utils operators** trigger **fd-nexus jobs** on Databricks:

```
Airflow DAG → orbit-utils operator → Databricks API → fd-nexus job
```

---

## Rule Organization

### Cursor Rules

```
.cursor/rules/
├── workspace.mdc (44 lines, alwaysApply: true)
├── fd-nexus.mdc (59 lines, globs: ["fd-nexus/**/*"])
└── orbit-utils.mdc (52 lines, globs: ["orbit-utils/**/*"])
```

**Pattern**: 
- **Workspace rule**: Always loaded, workspace overview
- **Project rules**: Loaded only when editing that project

**Why**: Precise control, minimal context footprint.

### Windsurf Rules

```
.windsurf/rules/
├── workspace.md (40 lines)
├── fd-nexus.md (55 lines)
└── orbit-utils.md (50 lines)
```

**Pattern**: All load for every session (no conditional loading).

**Why**: Must be extra concise (< 75 lines each).

### Claude Code

```
CLAUDE.md (109 lines, project root)
```

**Optional**:
```
fd-nexus/CLAUDE.md (override for fd-nexus directory)
orbit-utils/CLAUDE.md (override for orbit-utils directory)
```

---

## Memory-Bank Structure

### Two-Level Hierarchy

```
memory-bank/
├── INDEX.md (56 lines)        ← Workspace level
├── fd-nexus/
│   ├── INDEX.md (166 lines)   ← Project level
│   ├── project_overview.md
│   ├── components/
│   │   ├── transformations_overview.md
│   │   ├── readers_writers_overview.md
│   │   └── state_management_overview.md
│   ├── features/
│   │   ├── state-pattern/ (detailed specs)
│   │   └── external-api/
│   └── quick-refs/
│       ├── common_transformations.md (< 100 lines)
│       └── common_commands.md (< 100 lines)
└── orbit-utils/
    ├── INDEX.md
    ├── operators/
    └── workflows/
```

**Total**: 45+ files, ~20,000 lines

---

## Key Features

### 1. Hierarchical INDEX Pattern

**Workspace INDEX** (`memory-bank/INDEX.md` - 56 lines):

```markdown
# Workspace Memory-Bank Index

## Project Documentation

### fd-nexus (Spark Library)
**Index:** `fd-nexus/INDEX.md`
**Focus:** Transformations, readers/writers, state patterns

### orbit-utils (Airflow Operators)
**Index:** `orbit-utils/INDEX.md`
**Focus:** Databricks operators, workflow patterns

## Navigation by Task

| Task | Start With |
|------|------------|
| Implementing transformation | `fd-nexus/components/transformations_overview.md` |
| Building Airflow DAG | `orbit-utils/workflow-patterns.md` |
```

**Why**: Routes to project-specific INDEX, which has detailed navigation.

**Project INDEX** (`memory-bank/fd-nexus/INDEX.md` - 166 lines):

```markdown
# fd-nexus Memory-Bank Index

## Navigation by Task

| Task | Start With | Size |
|------|------------|------|
| Implementing transformation | `components/transformations_overview.md` | 450 lines |
| Creating ingest script | `scripts/ingest_patterns_overview.md` | 380 lines |
| Understanding state pattern | `features/state-pattern/implementation.md` | 1,400 lines |

## File Size Reference

### Lightweight (< 200 lines)
[List with line counts]

### Medium (200-500 lines)
[List with line counts]

### Large (500-1000 lines)
[List with line counts]

### Very Large (> 1000 lines)
[List with line counts]
```

**Why**: Project-specific navigation with full detail.

### 2. Four-Level Documentation Hierarchy

**Level 0: Index Files** (< 60 lines):
- `memory-bank/INDEX.md` (56 lines)
- `memory-bank/fd-nexus/INDEX.md` (166 lines)

**Level 1: Quick References** (< 100 lines):
- `quick-refs/common_transformations.md` (85 lines)
- `quick-refs/common_commands.md` (45 lines)

**Level 2: Component Overviews** (150-500 lines):
- `components/transformations_overview.md` (450 lines)
- `components/readers_writers_overview.md` (380 lines)

**Level 3: Deep References** (500-1000+ lines):
- `features/state-pattern/implementation.md` (1,400 lines)
- `features/external-api-transformation/adr.md` (1,200 lines)

**Why**: Progressive disclosure - start light, go deep when needed.

### 3. Transformation Registry Pattern

From `fd-nexus.mdc`:

```markdown
### Transformation Registry
50+ pre-built transformations in `transforms/registry.py`

## Anti-Patterns

- **ALWAYS** check 50+ existing transformations before creating new ones
```

From `quick-refs/common_transformations.md`:

```markdown
# Top 10 Most Common Transformations

1. `add_column` - Add new column with expression
2. `filter_rows` - Filter rows by condition
3. `join_tables` - Join with another DataFrame
...
```

**Why**: Prevents reinventing the wheel. AI checks registry first.

### 4. Glob-Based Conditional Loading (Cursor)

```yaml
# fd-nexus.mdc
---
globs: ["fd-nexus/**/*"]
---
```

**Result**: Rule loads only when editing fd-nexus files.

**Benefit**: Editing orbit-utils doesn't load fd-nexus rules → lower context.

### 5. Cross-Project Integration Documentation

From workspace rules:

```markdown
## Cross-Project Integration

orbit-utils operators invoke fd-nexus entry points:
- `fd_nexus_kafka_to_delta_stream`
- `fd_nexus_s3_to_delta_batch`
- `fd_nexus_landing_to_foundation`
```

**Why**: Developers need to understand how projects interact.

---

## Real-World Impact

**From production nexus implementation**:

### Before Context Engineering
- First-attempt success: 40%
- AI creates new transformations (50+ already exist!)
- Context bloated with full docs
- Team struggles to find patterns

### After Context Engineering
- First-attempt success: 75%
- AI checks transformation registry first
- Context stays under 20%
- New team members productive in days

**Setup time**: 4 hours initial, 30 min/month maintenance

---

## Adaptation Guide

### For Your Monorepo

**Copy**:
- Two-level INDEX hierarchy (workspace → project)
- Glob-based project rules (Cursor)
- 4-level documentation hierarchy
- Quick refs pattern

**Customize**:
- Project names and purposes
- Technology stacks
- Integration patterns
- Transformation/component patterns

### When to Use This Pattern

**Use monorepo pattern when**:
- 2+ distinct projects in same repo
- Projects have different tech stacks
- Projects interact but are separable
- Team size > 5 developers

**Don't use when**:
- Single project
- Small codebase (< 10,000 lines)
- Everything shares same patterns

---

## Key Lessons from Production

### 1. Hierarchical INDEX Scales

Single flat INDEX works for small projects. Monorepos need hierarchy:
- Workspace INDEX → Routes to projects
- Project INDEX → Routes to components

### 2. Quick Refs Save Context

100-line quick references for common tasks prevent loading 500-line comprehensive guides.

### 3. Glob Patterns are Critical for Monorepos

Without globs: All rules load for all files (context bloat).  
With globs: Only relevant rules load (lean context).

### 4. 4-Level Hierarchy Enables Progressive Disclosure

AI starts at index (60 lines), can navigate to 1,400-line spec if needed. But doesn't pre-load it.

### 5. Registry Pattern Prevents Duplication

Documenting "50+ transformations exist" prevents AI from creating duplicate functionality.

---

## Rule File Details

### workspace.mdc (44 lines)

**Sections**:
1. Package overview table
2. Cross-project integration
3. Memory-bank navigation
4. Code standards

**Pattern**: Minimal, points to memory-bank.

### fd-nexus.mdc (59 lines)

**Sections**:
1. Project context (3-layer architecture)
2. Key patterns (Script, Config, Factory, Registry)
3. Memory-bank references
4. Development commands
5. Anti-patterns (NEVER/ALWAYS rules)

**Pattern**: Project-specific constraints + memory-bank pointers.

### orbit-utils.mdc (52 lines)

**Similar to fd-nexus.mdc** but for Airflow operators.

---

## Setup Time

**Initial**: 4-5 hours
- Create workspace + project rules
- Build two-level INDEX hierarchy
- Write initial docs (5-10 files minimum)
- Count lines, categorize
- Test with real tasks

**Maintenance**: 30-60 minutes/month
- Update line counts
- Add new patterns
- Update for architecture changes
- Review and remove obsolete content

---

## Success Metrics

After setup:

- ✅ First-attempt success > 70%
- ✅ Context utilization < 25% before work
- ✅ AI navigates workspace → project → component correctly
- ✅ AI checks registry before creating new transformations
- ✅ Globe patterns work (Cursor)
- ✅ Team productivity high across both projects

---

## Files to Explore

Key patterns to review:

1. `memory-bank/INDEX.md` - Workspace-level navigation
2. `memory-bank/fd-nexus/INDEX.md` - Project-level navigation  
3. `.cursor/rules/workspace.mdc` - Minimal workspace rule
4. `.cursor/rules/fd-nexus.mdc` - Project-specific rule with globs
5. `memory-bank/fd-nexus/quick-refs/` - Quick reference pattern

---

## Scaling Beyond 2 Projects

**For 3-5 projects**:
- Same pattern, add more project rules
- Keep workspace rule minimal
- Each project gets own INDEX

**For 5+ projects**:
- Consider sub-INDEXes (by domain)
- Group related projects
- May need directory-level CLAUDE.md overrides

---

## Next Steps

1. Review the two-level hierarchy
2. Understand glob-based loading (Cursor)
3. Study the 4-level documentation pattern
4. Adapt for your monorepo
5. Test navigation with real tasks
6. Iterate based on team feedback

---

## References

- **Source inspiration**: FanDuel nexus production implementation
- See `../../docs/fundamentals/memory-bank-pattern.md` for pattern details
- See `../../docs/tools/cursor-guide.md` for glob patterns
- See `../../docs/best-practices/context-budgeting.md` for file sizing
