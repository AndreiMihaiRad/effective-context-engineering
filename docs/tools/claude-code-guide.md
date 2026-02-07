# Claude Code: Context Engineering Guide

> **Key insight**: Claude Code uses a file hierarchy for context (global → project → directory) and emphasizes workflow over static rules.

**Attribution**: Synthesized from nexus and data-api production CLAUDE.md implementations, plus Anthropic's context engineering guidelines.

---

## Quick Start

### 1. Initialize CLAUDE.md

```bash
# In your project root
claude /init
```

This generates a starter CLAUDE.md based on your codebase.

### 2. Review and Refine

Edit the generated CLAUDE.md:
- Remove inaccurate sections
- Add project-specific patterns
- Reference memory-bank for details
- Keep under 150 lines

### 3. Test It

```bash
claude
# Your CLAUDE.md is automatically loaded
```

---

## CLAUDE.md File Format and Hierarchy

### File Hierarchy

Claude Code searches for CLAUDE.md files in this order (highest priority last):

| Location | Scope | Priority | Use Case |
|----------|-------|----------|----------|
| `~/.claude/CLAUDE.md` | Global | Lowest | Personal preferences, universal tools |
| Project root `CLAUDE.md` | Project | Medium | Project-wide context, tech stack |
| Subdirectory `CLAUDE.md` | Directory | Highest | Component-specific overrides |

**Example**:
```
~/.claude/CLAUDE.md          # Your personal coding preferences
/project/CLAUDE.md           # Project-wide: React + FastAPI monorepo
/project/backend/CLAUDE.md   # Backend-specific: Python, FastAPI patterns
```

When working in `/project/backend/api.py`:
- Loads global CLAUDE.md
- Loads project CLAUDE.md
- Loads backend CLAUDE.md
- **backend/CLAUDE.md has highest priority**

### File Format

Plain Markdown (no frontmatter needed):

```markdown
# CLAUDE.md

This file provides guidance to Claude Code when working with this repository.

## Workspace Overview
[High-level description]

## Development Commands
[Common commands]

## Architecture
[Key patterns]

## Memory-Bank Navigation
Start with `memory-bank/INDEX.md` for detailed documentation.
```

### CLAUDE.md Loading in Monorepos

Understanding loading semantics prevents silent configuration failures:

| Direction | Behavior | When |
|-----------|----------|------|
| **Ancestor (UP)** | Walks UP from cwd, loads all ancestor CLAUDE.md files | At startup |
| **Descendant (DOWN)** | Lazy-loads subdirectory CLAUDE.md when files are touched | On demand |
| **Siblings (NEVER)** | Sibling directory CLAUDE.md files never load | Never |

```
monorepo/
├── CLAUDE.md                 # Always loads (ancestor)
├── packages/
│   ├── frontend/CLAUDE.md    # Lazy-loads when touching frontend/ files
│   └── backend/CLAUDE.md     # NEVER loads when working in frontend/
```

> See `docs/tools/claude-code-monorepo.md` for detailed monorepo guidance.

### WHY/WHAT/HOW Framework

Structure your CLAUDE.md for maximum effectiveness using this framework:

1. **WHY** (Purpose) — What is this project? What problem does it solve?
2. **WHAT** (Stack & Structure) — Tech stack, architecture, key directories
3. **HOW** (Commands & Constraints) — Build commands, testing, coding standards, anti-patterns

Place **critical instructions at the beginning and end** of CLAUDE.md — Claude pays most attention to these positions (attention optimization).

---

## Modular Rules (`.claude/rules/`)

Beyond CLAUDE.md, Claude Code supports modular rules with path-scoping via YAML frontmatter:

**`.claude/rules/frontend-standards.md`**:
```markdown
---
path: "src/frontend/**"
---

Use React Query for all data fetching.
Never use useEffect for API calls.
All components must have TypeScript props interfaces.
```

Rules in `.claude/rules/` are loaded based on the `path` frontmatter — they only apply when Claude is working on matching files. Rules without a `path` apply globally.

---

## Skills (`.claude/skills/`)

Skills are AI-discoverable capabilities that Claude automatically uses when relevant.

**Location**: `.claude/skills/` (fixed at project root, does NOT walk ancestors)

**Structure**: Each skill is a directory with a `SKILL.md` containing YAML frontmatter:

```markdown
---
name: code-review
description: Performs thorough code review with security and performance checks
model: opus
allowed-tools: ["Read", "Grep", "Glob"]
---

## Code Review Skill

When asked to review code:
1. Check for security vulnerabilities (OWASP Top 10)
2. Check for performance issues
3. Check for adherence to project patterns
...
```

**Priority hierarchy**: User skills (`~/.claude/skills/`) > Project skills (`.claude/skills/`)

Claude auto-discovers skills based on the task context — you don't need to explicitly invoke them.

> See `templates/claude/skills/` for skill templates.

---

## Agents (`.claude/agents/`)

Agents are autonomous sub-agents invoked via the **Task tool** (not bash). They have their own context and tool access.

**`.claude/agents/researcher.md`**:
```markdown
---
name: researcher
description: Deep codebase research and analysis
tools: ["Read", "Grep", "Glob", "WebSearch"]
model: sonnet
---

## Researcher Agent

You are a research specialist. When given a topic:
1. Systematically explore the codebase
2. Document all relevant files with line numbers
3. Return a concise summary (< 500 words)
```

**Key distinction**: Agents are invoked by Claude via the Task tool, not by users directly. Use commands for user-invoked workflows, skills for AI-discovered capabilities, and agents for autonomous sub-tasks.

> See `docs/tools/claude-code-advanced.md` for the Command → Skill → Agent architecture.

---

## Hooks

Hooks are shell commands that execute in response to Claude Code events:

| Event | When | Use Case |
|-------|------|----------|
| `PreToolUse` | Before a tool runs | Validate operations, block dangerous commands |
| `PostToolUse` | After a tool runs | Run linters after file writes, format code |
| `Notification` | On notifications | Send alerts to Slack, log events |
| `Stop` | When Claude stops | Cleanup, final validation |

**Exit codes**: `0` = success, `1` = error (blocks operation), `2` = success (output replaces tool result)

**Critical rule**: Never use LLMs as linters in hooks — hooks must be fast and deterministic.

Configure hooks in `.claude/settings.json`:
```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "command": "npx eslint --fix $CLAUDE_FILE_PATH"
    }]
  }
}
```

> See `docs/tools/claude-code-advanced.md` for hook examples and `docs/tools/claude-code-settings-reference.md` for configuration.

---

## Settings Overview

Claude Code settings follow a 4-tier hierarchy (highest priority first):

1. **CLI flags** — override everything
2. **Local** (`.claude/settings.local.json`) — personal, git-ignored
3. **Team** (`.claude/settings.json`) — shared, committed
4. **Global** (`~/.claude/settings.json`) — user-wide defaults

Key settings categories: permissions, hooks, MCP servers, model config, display/UX.

> See `docs/tools/claude-code-settings-reference.md` for the complete reference.

---

## Updated Context Management

### @ Syntax for File References

Reference files directly in prompts:

```
"Review @src/auth/login.ts for security issues"
"Compare @src/old-impl.ts with @src/new-impl.ts"
```

### ! Prefix for Shell Commands

Include shell output in context:

```
"Fix the failing tests shown in !npm test"
"The error is in !cat logs/error.log"
```

### Frequent Intentional Compaction (FIC)

Don't wait for context to fill up. Compact proactively at 40-60%:

```
"Compact our progress so far into a summary. Include:
- What's been completed
- Key decisions and rationale
- Remaining tasks
- Files modified"
```

Then `/clear` and continue with the compacted summary.

---

## Key Commands

### Essential Commands

| Command | Purpose | Usage |
|---------|---------|-------|
| `/init` | Generate starter CLAUDE.md | Run once per project |
| `/clear` | Reset context window | Between unrelated tasks |
| `/cost` | Check token usage | Monitor spending |
| `/add <path>` | Add files/dirs to context | Manual context loading |
| `/drop <path>` | Remove from context | Free up context |

### Extended Thinking

| Keyword | Effect | Use When |
|---------|--------|----------|
| `think` | Basic extended reasoning | Moderate complexity |
| `think hard` | More computation | Important decisions |
| `think harder` | Maximum depth | Architecture, complex debugging |

> **Note**: `ultrathink` is deprecated as of early 2026. Extended thinking depth is now controlled via the effort level in `/model` for Opus 4.6. Use `think`, `think hard`, or `think harder` instead.

**Example**:
```
"think harder about the best architecture for this feature"
```

### Custom Commands

Create reusable prompts in `.claude/commands/`:

**`.claude/commands/review.md`**:
```markdown
Review this code for:
- Security vulnerabilities
- Performance issues
- Best practice violations

Focus on the following file(s): $ARGUMENTS
```

**Usage**:
```bash
claude /review src/auth/login.ts
```

---

## Structured Workflow

Claude Code emphasizes a **three-phase workflow**:

### Phase 1: Research

**Goal**: Understand how the system works

**Commands**:
```bash
# Start fresh
claude /clear

# Research prompt
"Research how authentication works in this codebase. 
Create a research output file listing all relevant files."
```

**Output**: Research file with:
- Relevant file paths and line numbers
- Key architectural decisions
- Dependencies and relationships

### Phase 2: Planning

**Goal**: Specify every change before implementation

**Commands**:
```bash
# From research, create detailed plan
"Based on the research, create an implementation plan for adding OAuth support.
Include exact files, functions, and tests needed."
```

**Output**: Implementation plan with:
- Files to modify
- Code snippets showing changes
- Test cases to add
- Verification steps

### Phase 3: Implementation

**Goal**: Write code, maintain context under 40%

**Commands**:
```bash
# Execute plan
"Implement the OAuth support following the plan. 
Keep context utilization under 40%."
```

**Best practices during implementation**:
- Monitor context with `/cost`
- Use `/drop` for files no longer needed
- Use `/clear` between unrelated tasks
- Update plan as you go

---

## Context Management Strategies

### Intentional Compaction

When context approaches ~50% (don't wait until 80% — quality has already degraded by then). Use Frequent Intentional Compaction (FIC) at 40-60%:

```bash
"Compact the conversation into a progress file.
Include:
- Completed tasks
- Current state
- Pending tasks
- Key decisions
"
```

Save output to `progress.md`, then:
```bash
claude /clear
# Read progress.md and continue
```

### Structured Note-Taking

Create persistent memory outside context window:

```bash
"Write notes to NOTES.md about:
- Architecture decisions made
- Patterns to follow
- Issues encountered
"
```

Read back when needed:
```bash
"Read NOTES.md and continue with next task"
```

### Context Reset

Between distinct tasks:
```bash
claude /clear
```

This:
- Clears message history
- Clears file contents
- **Preserves** CLAUDE.md configuration
- Starts fresh context window

---

## Integration with Memory-Bank Pattern

### CLAUDE.md as Navigation Hub

**CLAUDE.md** (< 150 lines):
```markdown
# CLAUDE.md

## Workspace Overview
[Brief description - 50 lines max]

## Development Commands
[Common commands - 30 lines max]

## Architecture
[High-level only - 50 lines max]

## Memory-Bank Navigation
Start with `memory-bank/INDEX.md` for detailed documentation.

### Key Documentation
- `memory-bank/architecture/` - System design
- `memory-bank/components/` - Component specs
- `memory-bank/workflows/` - How-to guides

## Quick Reference
[Most common patterns - 50 lines max]
```

**memory-bank/** (same as Cursor/Windsurf):
- Detailed documentation
- Progressive disclosure
- Task-based navigation
- File size annotations

### Example from data-api (Production)

**CLAUDE.md** (266 lines):
- Workspace overview (50 lines)
- Development commands (40 lines)
- Architecture summary (60 lines)
- **Memory-bank navigation** (30 lines) ← Critical
- Technology stack (30 lines)
- Common tasks (40 lines)
- Critical constraints (16 lines)

**memory-bank/INDEX.md** (157 lines):
- Task-based navigation
- File size reference
- Quick start paths

**Result**: CLAUDE.md stays concise, detailed docs in memory-bank.

---

## Best Practices

### 1. Keep CLAUDE.md Under 150 Lines

```
❌ CLAUDE.md with 1,000 lines
✅ CLAUDE.md with ~120 lines + memory-bank references
```

Claude Code's system prompt consumes ~50 instructions, leaving ~100-150 for yours. Adherence drops significantly above 150 lines.

### 2. Use Spec-First Workflow

```
Always: Research → Plan → Implement
Never: Jump straight to coding
```

Better plans = less iteration = lower context usage.

### 3. Monitor Context Actively

```bash
# Check regularly
claude /cost

# Target: < 40% during implementation
```

### 4. Clear Context Between Tasks

```bash
# Finish feature A
claude /clear

# Start feature B (fresh context)
```

Don't carry accumulated context between unrelated tasks.

### 5. Reference Memory-Bank Heavily

```markdown
## Adding New Endpoint

For detailed guide, see `memory-bank/workflows/adding-endpoint.md`

Quick version:
1. Create TOML definition
2. Write SQL template
3. Validate and test
```

Surface overview in CLAUDE.md, details in memory-bank.

### 6. Use Extended Thinking for Complex Tasks

```
"think harder about the migration strategy for this schema change"
```

Invest thinking budget upfront for better outcomes. Use `/model` to set effort level for Opus 4.6.

---

## Common Pitfalls

### 1. Blindly Auto-Generating with /init

```
❌ Run /init and use the output as-is (often produces bloated, generic content)
✅ Start from a template, iterate based on failures. Use /init only for inspiration, then edit down to under 150 lines.
```

`/init` can generate useful starting content, but it typically produces files that are too long and too generic. Always start from a curated template and selectively incorporate `/init` suggestions.

### 2. Embedding Full Documentation

```
❌ CLAUDE.md (1,500 lines) with all API specs
✅ CLAUDE.md (120 lines) → memory-bank/api/
```

### 3. Skipping Research Phase

```
❌ "Implement OAuth support" (no research)
✅ "Research auth architecture, create research file, then plan"
```

Bad research → bad plan → wasted implementation time.

### 4. Not Monitoring Context

```
❌ Let context grow to 80%+ (quality already degraded)
✅ Monitor with /cost, compact at 50% using Frequent Intentional Compaction (FIC)
```

High context usage → degraded performance.

### 5. Forgetting /clear

```
❌ Single 200-turn conversation covering 5 features
✅ /clear between each feature
```

---

## Example Configurations

### Minimal Single-Project CLAUDE.md

```markdown
# CLAUDE.md

## Project Overview
FastAPI REST API for data access platform.

## Tech Stack
- Python 3.12
- FastAPI + Pydantic
- SQLAlchemy + Alembic
- pytest

## Development Commands
```bash
poetry install      # Setup
poetry run pytest   # Tests
poetry run uvicorn main:app --reload  # Run dev server
```

## Architecture
Hexagonal architecture:
- `domain/` - Business logic
- `adapters/` - External integrations
- `ports/` - Interface definitions

## Memory-Bank
Start with `memory-bank/INDEX.md` for detailed documentation.

## Quick Reference
- Adding endpoint: See `memory-bank/workflows/adding-endpoint.md`
- Testing: See `memory-bank/workflows/testing.md`
```

### Monorepo CLAUDE.md (nexus example)

```markdown
# CLAUDE.md

This file provides guidance to Claude Code for the Nexus workspace.

## Workspace Overview

| Package | Purpose | Python |
|---------|---------|--------|
| `fd-nexus/` | Spark library for Databricks | 3.10+ |
| `orbit-utils/` | Airflow operators | 3.12+ |

## Development Commands

### fd-nexus
```bash
cd fd-nexus
hatch run fmt       # Format
hatch run verify    # Lint + type check
hatch run test      # Tests
```

### orbit-utils
```bash
cd orbit-utils
hatch env run -- pytest tests/
```

## Architecture

### fd-nexus: Three-Layer Design
```
Script Layer → Component Layer → Utility Layer
(BaseJob)      (Readers/Writers)  (Spark utilities)
```

Key patterns:
- **Script Pattern**: `BaseJob → BaseIngestScript → SpecificScript`
- **Transformation Registry**: 50+ transformations in registry.py
- **Factory Pattern**: Readers, writers, state stores

### orbit-utils: Airflow Operators
Operators invoke fd-nexus entry points on Databricks.

## Memory-Bank Navigation

Start with `memory-bank/INDEX.md` for workspace documentation.

### Project Indexes
- `memory-bank/fd-nexus/INDEX.md` - Spark library docs
- `memory-bank/orbit-utils/INDEX.md` - Airflow operator docs

## Code Standards
- Formatting: Black-compatible (ruff)
- Type hints: Required for public APIs
- Docstrings: Google style
- Package management: hatch

## Quick Reference

### Adding Transformation (fd-nexus)
1. Inherit from `DataFrameTransform`
2. Define `required_params` and `default_params`
3. Implement `validate()` and `transform()`
4. Register in `TRANSFORM_REGISTRY`
5. Add tests

See `memory-bank/fd-nexus/components/transformations_overview.md` for details.
```

---

## Advanced Techniques

### Sub-Agent Pattern

For very complex tasks, use specialized contexts:

**Main conversation**:
```
"Launch sub-agent to research auth implementation.
Return summary of key files and patterns."
```

**Sub-agent** (new conversation):
```
# Does deep research, returns condensed summary
```

**Back to main**:
```
# Uses summary to create plan
# Never loads full research into main context
```

### Long-Horizon Tasks

For tasks spanning hours:

1. **Break into phases**: Research → Plan → Implement
2. **Write progress files**: After each phase
3. **Use /clear between phases**: Fresh context
4. **Maintain NOTES.md**: Architectural decisions persist

### Compaction Strategy

When approaching context limit:

```
"Create compaction file with:
- All modified file paths
- Key architectural decisions
- Remaining tasks
- Current implementation state

Use maximum recall, then refine for precision."
```

---

## Summary

| Aspect | Best Practice |
|--------|---------------|
| **File location** | Project root `CLAUDE.md` |
| **Format** | Plain Markdown |
| **Size** | < 150 lines |
| **Content** | Overview + memory-bank references |
| **Workflow** | Research → Plan → Implement |
| **Context management** | Monitor with `/cost`, clear with `/clear` |
| **Extended thinking** | Use for complex decisions |

---

## References

- [Using CLAUDE.md Files](https://www.claude.com/blog/using-claude-md-files) — Official guide
- [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — Anthropic research
- [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices) — Official best practices
- [Claude Code Skills](https://docs.anthropic.com/en/docs/claude-code/skills) — Skills documentation
- [Claude Code Hooks](https://docs.anthropic.com/en/docs/claude-code/hooks) — Hooks documentation
- See `docs/tools/claude-code-advanced.md` for advanced patterns (skills, agents, hooks, MCP)
- See `docs/tools/claude-code-settings-reference.md` for complete settings reference
- See `docs/tools/claude-code-monorepo.md` for monorepo-specific guidance
- See `docs/fundamentals/context-engineering.md` for underlying principles
- See `docs/fundamentals/memory-bank-pattern.md` for the memory-bank pattern
- See `templates/claude/` for ready-to-use templates
- See `examples/` for real-world configurations
