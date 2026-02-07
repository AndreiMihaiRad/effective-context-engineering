# Claude Code: Settings Reference

> **Key insight**: Claude Code settings follow a 4-tier hierarchy that enables team-shared defaults with personal overrides.

**Attribution**: Adapted from claude-code-best-practices repository, Anthropic official docs, and production configurations.

---

## Settings Hierarchy

Claude Code resolves settings from multiple sources. When the same key appears at multiple levels, higher-priority wins.

| Priority | Location | Scope | Git-tracked? |
|----------|----------|-------|--------------|
| 1 (highest) | CLI flags (`--allowedTools`, etc.) | Invocation | No |
| 2 | `.claude/settings.local.json` | Project (personal) | No (git-ignored) |
| 3 | `.claude/settings.json` | Project (team) | Yes |
| 4 | `~/.claude/settings.json` | User-wide | No |
| 5 (lowest) | Managed policy (enterprise) | Organization | Managed externally |

**How merging works**:
- Permission arrays (`allow`, `deny`) are **merged** across tiers (union of all entries).
- Scalar values (e.g., model selection) use the **highest-priority** tier that defines them.
- A `deny` at any tier blocks the tool regardless of `allow` at other tiers.

**Example**: Your team shares `.claude/settings.json` with safe defaults. You add personal API keys and model preferences in `.claude/settings.local.json` (git-ignored). CLI flags override both for one-off invocations.

---

## Permissions

Permissions control which tools Claude Code can use and whether it asks for confirmation.

### Permission Levels

| Level | Behavior |
|-------|----------|
| `allow` | Tool runs without asking for confirmation |
| `ask` | Prompt user for confirmation (default for most tools) |
| `deny` | Block tool entirely, Claude cannot use it |

### Tool Permission Syntax

Permissions use a `ToolName(argument_pattern)` syntax:

| Permission String | Matches |
|-------------------|---------|
| `Bash(*)` | All bash commands |
| `Bash(git commit:*)` | `git commit` with any arguments |
| `Bash(git diff:*)` | `git diff` with any arguments |
| `Bash(npm test)` | Exactly `npm test` (no extra args) |
| `Bash(npm test:*)` | `npm test` with any arguments |
| `Bash(poetry run pytest:*)` | pytest via poetry with any arguments |
| `Edit` | All file editing operations |
| `Write` | All file creation operations |
| `Read(*)` | Read any file |
| `WebFetch(*)` | Fetch any URL |
| `mcp__serverName__toolName` | Specific MCP server tool |
| `mcp__context7__*` | All tools from the context7 MCP server |

### Example Permissions Configuration

```json
{
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(npm test:*)",
      "Bash(npm run lint:*)",
      "Bash(npx tsc:*)",
      "Read(*)",
      "Edit",
      "mcp__context7__resolve-library-id",
      "mcp__context7__query-docs"
    ],
    "deny": [
      "Bash(curl:*)",
      "Bash(wget:*)",
      "Bash(rm -rf:*)",
      "Bash(sudo:*)",
      "Bash(chmod:*)",
      "Bash(git push --force:*)",
      "Bash(git reset --hard:*)"
    ]
  }
}
```

**Notes**:
- Tools not listed in `allow` or `deny` default to `ask` (interactive confirmation).
- The colon `:` in `Bash(git commit:*)` separates the command prefix from its arguments.
- Patterns are matched left-to-right; `Bash(git:*)` allows all git subcommands.

---

## Hooks Configuration

Hooks let you run custom scripts at specific points in Claude Code's execution lifecycle. They enable linting, validation, logging, and custom guardrails.

### Hook Events

| Event | Trigger | Use Cases |
|-------|---------|-----------|
| `PreToolUse` | Before a tool executes | Lint checks, validation, blocking |
| `PostToolUse` | After a tool executes | Post-processing, logging, notifications |
| `Notification` | When Claude produces a notification | Alerts, custom notification routing |
| `Stop` | When Claude finishes a turn | Summary generation, cleanup |
| `SubagentStop` | When a subagent finishes | Subagent output validation |

### Hook Structure

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "python .claude/hooks/lint-check.py"
      },
      {
        "matcher": "Bash",
        "command": ".claude/hooks/validate-command.sh"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "python .claude/hooks/post-edit-format.py"
      }
    ],
    "Notification": [
      {
        "matcher": "",
        "command": "python .claude/hooks/send-notification.py"
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "command": ".claude/hooks/on-stop.sh"
      }
    ]
  }
}
```

### How Hooks Work

- The `matcher` field is a regex tested against the tool name. Empty string matches all.
- The hook `command` receives tool input via **stdin** as JSON.
- Hook exit codes control behavior:
  - **Exit 0**: Proceed normally. Any stdout is added as context.
  - **Exit 2** (PreToolUse only): Block the tool call. Stdout becomes the reason shown to Claude.
  - **Other non-zero**: Error logged but execution continues.

### Example Hook Script

`.claude/hooks/lint-check.py`:
```python
#!/usr/bin/env python3
"""PreToolUse hook: lint files before Write/Edit."""
import json
import subprocess
import sys

input_data = json.load(sys.stdin)
file_path = input_data.get("tool_input", {}).get("file_path", "")

if file_path.endswith(".py"):
    result = subprocess.run(
        ["ruff", "check", "--select", "E,F", file_path],
        capture_output=True, text=True
    )
    if result.returncode != 0:
        print(f"Lint errors found:\n{result.stdout}")
        sys.exit(2)  # Block the tool call

sys.exit(0)  # Allow the tool call
```

---

## MCP Configuration

Model Context Protocol (MCP) servers extend Claude Code with additional tools and data sources.

### Configuration Format

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"],
      "env": {}
    },
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"],
      "env": {}
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "postgresql://user:pass@localhost:5432/db"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": [
        "-y", "@modelcontextprotocol/server-filesystem",
        "/path/to/allowed/directory"
      ],
      "env": {}
    }
  }
}
```

### MCP Server Fields

| Field | Required | Description |
|-------|----------|-------------|
| `command` | Yes | Executable to launch the server |
| `args` | Yes | Arguments passed to the command |
| `env` | No | Environment variables for the server process |

### Where to Configure MCP

- **Global** (`~/.claude/settings.json`): Servers available in all projects (e.g., context7, sequential-thinking).
- **Project** (`.claude/settings.json`): Servers specific to a project (e.g., project database, custom APIs).
- **Personal** (`.claude/settings.local.json`): Servers with personal credentials (e.g., database with your credentials).

### MCP Tool Permissions

MCP tools follow the `mcp__<serverName>__<toolName>` naming convention:

```json
{
  "permissions": {
    "allow": [
      "mcp__context7__resolve-library-id",
      "mcp__context7__query-docs",
      "mcp__sequential-thinking__sequentialthinking"
    ]
  }
}
```

---

## Model Configuration

### Selecting Models

Use the `/model` command interactively, or set via environment variables:

```bash
# Set the primary model
export ANTHROPIC_MODEL="claude-opus-4-6"

# Or use the alias
claude --model claude-opus-4-6
```

### Effort Levels (Opus 4.6)

Opus 4.6 supports configurable effort levels via the `/model` command:

| Level | Behavior | Use When |
|-------|----------|----------|
| Low | Faster responses, less reasoning | Simple tasks, quick edits |
| Medium | Balanced (default) | Most development work |
| High | Maximum reasoning depth | Architecture decisions, complex debugging |

**Note**: The `ultrathink` keyword is deprecated as of early 2026. Use `think`, `think hard`, or `think harder` inline, or configure the effort level via `/model` for persistent behavior.

---

## Display and UX

### Status Line

Customize the status line shown during processing:

```json
{
  "statusLine": {
    "enabled": true
  }
}
```

### Spinner Verbs

Customize the loading spinner text:

```json
{
  "spinnerVerbs": ["Thinking", "Analyzing", "Processing"]
}
```

### Theme

Claude Code follows your terminal theme. Additional display preferences:

```json
{
  "theme": "dark",
  "verbose": false
}
```

---

## Environment Variables

Environment variables configure Claude Code behavior at the system level. Set them in your shell profile or `.env` file.

### Core Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `ANTHROPIC_API_KEY` | API key for Anthropic | Required |
| `ANTHROPIC_MODEL` | Model to use | `claude-sonnet-4-20250514` |
| `CLAUDE_CODE_MAX_TURNS` | Max agentic turns in non-interactive mode | `25` |
| `CLAUDE_CODE_MAX_TOOL_USE_TOKENS` | Token budget for tool use | Model-dependent |
| `CLAUDE_CODE_BUDGET_TOKENS` | Total token budget for a session | Unlimited |

### Provider Selection

| Variable | Description |
|----------|-------------|
| `CLAUDE_CODE_USE_BEDROCK` | Use AWS Bedrock as provider (`1` to enable) |
| `CLAUDE_CODE_USE_VERTEX` | Use Google Vertex AI as provider (`1` to enable) |
| `ANTHROPIC_BASE_URL` | Custom API base URL (for proxies) |

### AWS Bedrock Variables

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_BEDROCK_BASE_URL` | Bedrock endpoint URL |
| `AWS_REGION` | AWS region |
| `AWS_ACCESS_KEY_ID` | AWS access key |
| `AWS_SECRET_ACCESS_KEY` | AWS secret key |
| `AWS_SESSION_TOKEN` | AWS session token (temporary credentials) |
| `AWS_PROFILE` | AWS profile name |

### Google Vertex AI Variables

| Variable | Description |
|----------|-------------|
| `ANTHROPIC_VERTEX_PROJECT_ID` | GCP project ID |
| `CLOUD_ML_REGION` | GCP region |
| `ANTHROPIC_VERTEX_BASE_URL` | Vertex endpoint URL |

### Sandbox and Security

| Variable | Description | Default |
|----------|-------------|---------|
| `CLAUDE_CODE_SANDBOX_TYPE` | Sandbox type: `docker`, `none` | `none` |
| `CLAUDE_CODE_DISABLE_NONINTERACTIVE` | Disable non-interactive/headless mode | `0` |

### MCP Variables

| Variable | Description |
|----------|-------------|
| `MCP_TIMEOUT` | Timeout for MCP server connections (ms) |
| `MCP_TOOL_TIMEOUT` | Timeout for individual MCP tool calls (ms) |

### Telemetry and Debugging

| Variable | Description |
|----------|-------------|
| `CLAUDE_CODE_TELEMETRY_DISABLED` | Disable telemetry (`1` to disable) |
| `CLAUDE_CODE_DEBUG` | Enable debug logging (`1` to enable) |
| `CLAUDE_CODE_LOG_LEVEL` | Log level: `debug`, `info`, `warn`, `error` |

### Non-Interactive / CI Mode

| Variable | Description |
|----------|-------------|
| `CLAUDE_CODE_HEADLESS` | Run in headless mode for CI/CD (`1` to enable) |
| `CLAUDE_CODE_ENTRYPOINT` | Initial prompt for headless mode |

---

## Complete Example: Team Settings

### `.claude/settings.json` (committed to repo)

```json
{
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git branch:*)",
      "Bash(git checkout:*)",
      "Bash(git switch:*)",
      "Bash(npm test:*)",
      "Bash(npm run lint:*)",
      "Bash(npm run build:*)",
      "Bash(npx tsc:*)",
      "Bash(npx prettier:*)",
      "Read(*)",
      "Edit",
      "Write"
    ],
    "deny": [
      "Bash(curl:*)",
      "Bash(wget:*)",
      "Bash(rm -rf:*)",
      "Bash(sudo:*)",
      "Bash(chmod 777:*)",
      "Bash(git push --force:*)",
      "Bash(git reset --hard:*)",
      "Bash(git clean -f:*)",
      "Bash(npm publish:*)"
    ]
  },
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "python .claude/hooks/lint-check.py"
      }
    ],
    "PostToolUse": [],
    "Notification": [],
    "Stop": []
  },
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"],
      "env": {}
    }
  }
}
```

### `.claude/settings.local.json` (personal, git-ignored)

```json
{
  "permissions": {
    "allow": [
      "Bash(docker:*)",
      "Bash(poetry run pytest:*)",
      "mcp__sequential-thinking__sequentialthinking",
      "mcp__postgres__query"
    ]
  },
  "mcpServers": {
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"],
      "env": {}
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_CONNECTION_STRING": "postgresql://dev:dev@localhost:5432/myapp"
      }
    }
  }
}
```

### `~/.claude/settings.json` (global user defaults)

```json
{
  "permissions": {
    "allow": [
      "Bash(git status)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Read(*)"
    ],
    "deny": [
      "Bash(sudo:*)",
      "Bash(rm -rf /)"
    ]
  },
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp@latest"],
      "env": {}
    }
  }
}
```

---

## Setting Up a New Project

### Step 1: Create Team Settings

```bash
mkdir -p .claude
```

Create `.claude/settings.json` with permissions appropriate for your project's toolchain (see Complete Example above).

### Step 2: Add to .gitignore

```bash
# In your .gitignore
.claude/settings.local.json
```

### Step 3: Create Personal Overrides (Optional)

Create `.claude/settings.local.json` for personal MCP servers, extra permissions, or credentials.

### Step 4: Verify

```bash
claude
# Type: /permissions
# Verify allowed and denied tools match expectations
```

---

## Troubleshooting

### Permission Denied for a Tool

A `deny` at **any** tier blocks the tool. Check all tiers:

```bash
# Check project team settings
cat .claude/settings.json

# Check personal settings
cat .claude/settings.local.json

# Check global settings
cat ~/.claude/settings.json
```

### MCP Server Not Connecting

1. Verify the server command works standalone: `npx -y @upstash/context7-mcp@latest`
2. Check environment variables are set correctly in the `env` field.
3. Increase timeout: `export MCP_TIMEOUT=30000`
4. Enable debug logging: `export CLAUDE_CODE_DEBUG=1`

### Hooks Not Running

1. Verify the hook script is executable: `chmod +x .claude/hooks/lint-check.py`
2. Check the `matcher` regex matches the intended tool name.
3. Test the hook script standalone with sample JSON on stdin.

---

## References

- [Claude Code Overview](https://docs.anthropic.com/en/docs/claude-code/overview)
- [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Claude Code Settings Documentation](https://docs.anthropic.com/en/docs/claude-code/settings)
- See `docs/tools/claude-code-guide.md` for CLAUDE.md and workflow guidance
- See `docs/fundamentals/context-engineering.md` for underlying principles
- See `docs/best-practices/rule-writing.md` for rule writing guidance
- See `templates/claude/` for ready-to-use templates
