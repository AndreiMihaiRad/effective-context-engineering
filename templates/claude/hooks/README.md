# Claude Code Hooks

Hooks are shell commands that execute in response to Claude Code events. They enable automated validation, linting, and notifications.

## Setup

Hooks are configured in `.claude/settings.json` under the `hooks` key.

## Hook Events

| Event | When | Use Case |
|-------|------|----------|
| `PreToolUse` | Before a tool runs | Validate operations, block dangerous commands |
| `PostToolUse` | After a tool runs | Run linters, format code, validate output |
| `Notification` | On notifications | Send alerts to Slack, log events |
| `Stop` | When Claude stops | Cleanup, final validation |
| `SubagentStop` | When a sub-agent stops | Aggregate results |

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success — proceed normally |
| `1` | Error — block the operation |
| `2` | Success — output replaces tool result |

## Environment Variables

Hooks receive these environment variables:

| Variable | Description |
|----------|-------------|
| `CLAUDE_TOOL_NAME` | Name of the tool being used |
| `CLAUDE_FILE_PATH` | Path to the file being operated on |
| `CLAUDE_TOOL_INPUT` | JSON string of tool input parameters |
| `CLAUDE_TOOL_OUTPUT` | JSON string of tool output (PostToolUse only) |

## Example: PostToolUse Linting

**`.claude/settings.json`**:
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "command": "python .claude/hooks/lint-check.py"
      }
    ]
  }
}
```

**`.claude/hooks/lint-check.py`**:
```python
#!/usr/bin/env python3
"""Run linter on files after Claude writes/edits them."""

import json
import os
import subprocess
import sys

file_path = os.environ.get("CLAUDE_FILE_PATH", "")

if not file_path:
    sys.exit(0)

# Run linter based on file extension
if file_path.endswith((".ts", ".tsx", ".js", ".jsx")):
    result = subprocess.run(
        ["npx", "eslint", "--fix", file_path],
        capture_output=True, text=True
    )
    if result.returncode != 0:
        print(f"ESLint issues in {file_path}:")
        print(result.stdout)
        sys.exit(1)

elif file_path.endswith(".py"):
    result = subprocess.run(
        ["ruff", "check", "--fix", file_path],
        capture_output=True, text=True
    )
    if result.returncode != 0:
        print(f"Ruff issues in {file_path}:")
        print(result.stdout)
        sys.exit(1)

sys.exit(0)
```

## Example: PreToolUse Validation

**`.claude/settings.json`**:
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "command": "python .claude/hooks/validate-command.py"
      }
    ]
  }
}
```

**`.claude/hooks/validate-command.py`**:
```python
#!/usr/bin/env python3
"""Block dangerous bash commands."""

import json
import os
import sys

tool_input = json.loads(os.environ.get("CLAUDE_TOOL_INPUT", "{}"))
command = tool_input.get("command", "")

BLOCKED_PATTERNS = [
    "rm -rf /",
    "DROP TABLE",
    "DROP DATABASE",
    "git push --force",
    "git reset --hard",
]

for pattern in BLOCKED_PATTERNS:
    if pattern.lower() in command.lower():
        print(f"BLOCKED: Command contains dangerous pattern '{pattern}'")
        sys.exit(1)

sys.exit(0)
```

## Critical Rule

**Never use LLMs as linters in hooks.** Hooks must be:
- **Fast** — they run on every tool use
- **Deterministic** — same input always produces same output
- **Lightweight** — no network calls to AI APIs

Use traditional tools (ESLint, Ruff, Prettier) in hooks, not AI.

## References

- See `docs/tools/claude-code-advanced.md` for advanced hook patterns
- See `docs/tools/claude-code-settings-reference.md` for settings configuration
- See `templates/claude/settings.json.template` for settings template
