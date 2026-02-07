# Claude Code Templates

Ready-to-use templates for Claude Code context configuration.

## Files Included

| Template | Purpose |
|----------|---------|
| `CLAUDE.md.template` | Project-level context file (WHY/WHAT/HOW framework) |
| `settings.json.template` | Team-shared settings (permissions, hooks, MCP) |
| `settings.local.json.template` | Personal settings (git-ignored) |
| `skills/example-skill/SKILL.md.template` | AI-discoverable skill template |
| `agents/example-agent.md.template` | Autonomous sub-agent template |
| `commands/example-command.md.template` | User-invoked command template |
| `hooks/README.md` | Hooks setup guide with examples |
| `rules/example-rule.md.template` | Modular rule with path-scoping |
| `rpi/` | Research-Plan-Implement workflow templates |

## Quick Start

### 1. Create CLAUDE.md

```bash
# Copy template
cp templates/claude/CLAUDE.md.template CLAUDE.md

# Fill in placeholders, keeping under 150 lines
# Use WHY/WHAT/HOW framework:
#   - WHY: Critical constraints (beginning of file)
#   - WHAT: Stack, architecture, structure
#   - HOW: Commands, standards, navigation
```

### 2. Create Settings

```bash
# Team settings (commit to repo)
mkdir -p .claude
cp templates/claude/settings.json.template .claude/settings.json

# Personal settings (add .claude/settings.local.json to .gitignore)
cp templates/claude/settings.local.json.template .claude/settings.local.json
```

### 3. (Optional) Add Skills, Commands, Agents

```bash
# Skills — AI-discovered capabilities
mkdir -p .claude/skills/code-review
cp templates/claude/skills/example-skill/SKILL.md.template .claude/skills/code-review/SKILL.md

# Commands — user-invoked prompts
mkdir -p .claude/commands
cp templates/claude/commands/example-command.md.template .claude/commands/review.md

# Agents — autonomous sub-agents
mkdir -p .claude/agents
cp templates/claude/agents/example-agent.md.template .claude/agents/researcher.md

# Rules — modular path-scoped rules
mkdir -p .claude/rules
cp templates/claude/rules/example-rule.md.template .claude/rules/frontend.md
```

### 4. (Optional) Set Up RPI Workflow

```bash
# Copy Research-Plan-Implement commands
cp -r templates/claude/rpi/commands/rpi .claude/commands/

# Copy senior engineer agent
cp templates/claude/rpi/agents/senior-engineer.md.template .claude/agents/senior-engineer.md
```

### 5. Create Memory-Bank

```bash
mkdir -p memory-bank
cp templates/memory-bank/INDEX.md.template memory-bank/INDEX.md
```

## Template Details

### CLAUDE.md.template

Uses the **WHY/WHAT/HOW framework**:
- **WHY** — Critical constraints at beginning (highest attention)
- **WHAT** — Stack, architecture, structure
- **HOW** — Commands, standards, navigation
- **WHY again** — Critical reminders at end (also high attention)

**Target size**: < 150 lines (Claude Code's system prompt consumes ~50 instructions)

### Settings Hierarchy

```
CLI flags > .claude/settings.local.json > .claude/settings.json > ~/.claude/settings.json
```

- `settings.json` — Team defaults (permissions, hooks, MCP)
- `settings.local.json` — Personal overrides (git-ignored)

### Command → Skill → Agent Architecture

Three tiers of progressive automation:

| Tier | Location | Invoked By | Use Case |
|------|----------|------------|----------|
| **Commands** | `.claude/commands/` | User (`/command-name`) | Direct, user-driven tasks |
| **Skills** | `.claude/skills/` | Claude (auto-discovered) | AI-discovered capabilities |
| **Agents** | `.claude/agents/` | Claude (via Task tool) | Autonomous sub-tasks |

## File Hierarchy

Claude Code searches for CLAUDE.md in this order:

| Location | Scope | Priority |
|----------|-------|----------|
| `~/.claude/CLAUDE.md` | Global | Lowest |
| `/project/CLAUDE.md` | Project | Medium |
| `/project/subdir/CLAUDE.md` | Directory | Highest |

## Best Practices

### 1. Keep CLAUDE.md Under 150 Lines

Adherence drops significantly above 150 lines. Move details to memory-bank/.

### 2. Start from Template, Not /init

`/init` produces generic, bloated content. Start from this template and selectively incorporate `/init` suggestions.

### 3. Place Critical Constraints First and Last

Claude pays most attention to the beginning and end of CLAUDE.md.

### 4. Use Settings for Permissions

Configure `allow`/`deny` in settings.json so Claude can run safe commands without asking.

## References

- See `docs/tools/claude-code-guide.md` for comprehensive guide
- See `docs/tools/claude-code-advanced.md` for advanced patterns
- See `docs/tools/claude-code-settings-reference.md` for settings reference
- See `docs/tools/claude-code-monorepo.md` for monorepo guidance
- See `docs/fundamentals/memory-bank-pattern.md` for memory-bank details
- See `examples/simple-project/` for a working example
