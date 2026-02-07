# RPI Workflow Templates

Research → Plan → Implement workflow for Claude Code.

## Overview

The RPI workflow breaks complex tasks into three reviewable phases:
1. **Research** — understand the system before changing it
2. **Plan** — specify every change before writing code
3. **Implement** — execute the plan with context management

## Setup

### 1. Copy Commands

```bash
# From your project root
cp -r templates/claude/rpi/commands/rpi .claude/commands/
```

### 2. (Optional) Copy Agent

```bash
cp templates/claude/rpi/agents/senior-engineer.md.template .claude/agents/senior-engineer.md
```

### 3. Edit Templates

Remove `.template` extensions and customize for your project.

## Usage

### Research Phase

```bash
# Start research
/rpi:research "authentication system"

# Review the output
# Human reviews research-authentication-system.md
```

### Plan Phase

```bash
# Create plan from research
/rpi:plan "authentication system"

# Review the plan
# Human reviews plan-authentication-system.md
```

### Implementation Phase

```bash
# Implement the plan
/rpi:implement "authentication system"

# Review the changes
# Human reviews code changes against plan
```

## Human Leverage Hierarchy

Where human review has the most impact:

1. **Research reviews** (highest leverage) — catching wrong assumptions early saves 10x
2. **Plan reviews** — catching wrong approaches saves 5x
3. **Code reviews** (lowest leverage) — catching bugs saves 1x

Invest review time accordingly: spend MORE time reviewing research and plans, LESS on code.

## Files

| File | Purpose |
|------|---------|
| `commands/rpi/research.md.template` | Research phase command |
| `commands/rpi/plan.md.template` | Planning phase command |
| `commands/rpi/implement.md.template` | Implementation phase command |
| `agents/senior-engineer.md.template` | Expert agent for complex tasks |

## References

- See `docs/workflow/spec-first-development.md` for methodology details
- See `docs/tools/claude-code-guide.md` for Claude Code basics
- See `docs/tools/claude-code-advanced.md` for advanced patterns
