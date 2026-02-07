# Claude Code: Advanced Patterns

> **Key insight**: Claude Code's 2026 feature set includes skills, agents, hooks, and MCP servers that enable sophisticated automation workflows beyond basic CLAUDE.md configuration.

**Attribution**: Synthesized from Anthropic official documentation, HumanLayer production patterns, Shipyard workflow implementations, ClaudeLog analysis, and real-world engineering team deployments.

---

## Command-Skill-Agent Architecture

Three-tier system for progressive disclosure. Each tier adds autonomy and complexity.

### Tier 1: Commands (User-Invoked)

**Location**: `.claude/commands/` -- user runs `/command-name` with `$ARGUMENTS` substituted.

**`.claude/commands/weather.md`**:
```markdown
Look up the current weather for the city: $ARGUMENTS
Report the temperature, conditions, and any weather alerts.
Use the fetch tool to query a weather API.
```

**Usage**: `claude /weather "San Francisco"`

Key traits: explicit invocation, simple markdown, no metadata, best for repeatable user-triggered tasks.

### Tier 2: Skills (AI-Discovered)

**Location**: `.claude/skills/` (project) or `~/.claude/skills/` (user, higher priority)

Skills are capabilities Claude discovers and uses autonomously. YAML frontmatter tells Claude when to apply them.

**`.claude/skills/weather-lookup.md`**:
```yaml
---
name: weather-lookup
description: Look up current weather conditions for any city worldwide
model: opus-4-6
context:
  - file: .env.example
    description: Contains WEATHER_API_KEY template
allowed-tools:
  - WebFetch
  - Bash
---
# Weather Lookup
When the user asks about weather conditions for a location:
1. Use the weather API endpoint: https://api.weatherapi.com/v1/current.json
2. Include the API key from environment variable WEATHER_API_KEY
3. Parse the JSON response and report: temperature, conditions, humidity, wind, alerts

## Error Handling
- Missing API key: inform user to set WEATHER_API_KEY
- City not found: suggest similar city names
- API down: fall back to web search
```

Key traits: auto-discovered, YAML frontmatter (`name`, `description`, `model`, `context`, `allowed-tools`), no user invocation needed.

### Tier 3: Agents (Autonomous Sub-Agents)

**Location**: `.claude/agents/` -- isolated sub-agents invoked via the Task tool (NOT bash).

**`.claude/agents/weather-analyst.md`**:
```yaml
---
name: weather-analyst
description: Comprehensive weather analysis with forecasts, historical data, and travel advice
tools: [WebFetch, WebSearch, Bash, Read, Write]
model: opus-4-6
color: blue
skills: [weather-lookup]
---
# Weather Analyst Agent
You are a weather analysis specialist. When invoked, perform comprehensive analysis:
1. **Current Conditions**: Fetch real-time weather data
2. **Forecast**: Get 5-day forecast from weather API
3. **Historical Context**: Search for historical averages for this time of year
4. **Severe Weather**: Check for any active warnings or watches
5. **Travel Advisory**: Provide travel recommendations based on conditions

Write analysis to `weather-report-{city}-{date}.md` with: Executive Summary,
Current Conditions, 5-Day Forecast (table), Historical Comparison, Recommendations.
```

**Invocation**: `Launch the weather-analyst agent to analyze conditions in Tokyo for next week.`

### Architecture Summary

| Tier | Location | Triggered By | Metadata | Isolation |
|------|----------|-------------|----------|-----------|
| Command | `.claude/commands/` | User (`/name`) | None | Same context |
| Skill | `.claude/skills/` | AI auto-discovery | YAML frontmatter | Same context |
| Agent | `.claude/agents/` | Task tool | YAML frontmatter | Own context |

---

## RPI Workflow Implementation

The Research-Plan-Implement pattern formalized as a command chain with structured output files.

### Command: /rpi:research

**`.claude/commands/rpi:research.md`**:
```markdown
## Research Phase
Research the following topic thoroughly: $ARGUMENTS

### Instructions
1. Identify all relevant files using Grep and Glob
2. Read each relevant file and understand the patterns
3. Map dependencies and relationships between components
4. Identify existing tests and their coverage

### Output
Create `rpi-output/research-{timestamp}.md` with:
- **Scope**: What was researched and why
- **Relevant Files**: Every file with path, line count, and purpose
- **Architecture**: How the relevant components connect
- **Patterns**: Existing patterns that must be followed
- **Dependencies**: External and internal dependencies
- **Risks**: Potential issues or conflicts
- **Test Coverage**: Existing tests and gaps
Do NOT propose solutions yet. This is pure research.
```

### Command: /rpi:plan

**`.claude/commands/rpi:plan.md`**:
```markdown
## Planning Phase
Create a detailed implementation plan. Read the most recent research file in `rpi-output/`.
Task to plan: $ARGUMENTS

Think harder about the implementation approach. Consider: files to modify vs create,
order of operations, backward compatibility, test strategy.

### Output
Create `rpi-output/plan-{timestamp}.md` with:
- **Objective**: One-sentence summary
- **Changes**: Ordered file modifications (exact functions/classes, what changes, why)
- **New Files**: Files to create with their purpose
- **Test Plan**: Specific tests to write
- **Verification Steps**: How to confirm it works
- **Rollback**: How to undo if something goes wrong
- **Estimated Complexity**: Low / Medium / High with justification
Present the plan for human review before proceeding.
```

### Command: /rpi:implement

**`.claude/commands/rpi:implement.md`**:
```markdown
## Implementation Phase
Execute the plan. Read the most recent plan in `rpi-output/`. Additional: $ARGUMENTS

1. Follow the plan exactly. Deviate only with explanation.
2. Implement in the specified order. Run tests after each file change.
3. Keep context under 40%. At 50%, checkpoint to `rpi-output/progress-{timestamp}.md`
   and inform user to `/clear` and resume.
4. After all changes, run full verification steps from the plan.
5. Update plan with [x] completed / [ ] remaining items and any deviations.
```

### Senior-Engineer Agent for Complex RPI

**`.claude/agents/senior-engineer.md`**:
```yaml
---
name: senior-engineer
description: Senior engineer agent for deep research, planning, and code review
tools: [Read, Glob, Grep, Bash, Write]
model: opus-4-6
color: green
skills: []
---
# Senior Engineer Agent
You are a senior software engineer performing deep technical analysis.
- Be thorough: check edge cases, error handling, security implications
- Be opinionated: recommend the best approach, not all approaches
- Be practical: consider maintenance burden and team skill level
- Question assumptions if something seems off
When researching: read broadly then deeply. Start with structure, drill into specifics.
When reviewing plans: evaluate correctness, completeness, simplicity, testability, risk.
```

**Usage**: `Launch the senior-engineer agent to research the auth system and find all session token validation points.`

---

## Hooks System

Hooks run scripts at specific points in Claude Code's execution. They must be fast and deterministic -- never use LLMs as linters.

### Hook Events

| Event | When It Fires | Common Use |
|-------|--------------|------------|
| `PreToolUse` | Before any tool executes | Block dangerous operations |
| `PostToolUse` | After any tool completes | Lint, format, validate |
| `Notification` | Claude sends a notification | Custom alerting |
| `Stop` | Claude finishes responding | Post-completion checks |
| `SubagentStop` | Sub-agent finishes | Validate sub-agent output |
| `PreCompact` / `PostCompact` | Before/after compaction | Save/restore state |
| `PreToolUse:Read` | Before Read call | Access control |
| `PreToolUse:Write` | Before Write call | Path validation |
| `PreToolUse:Bash` | Before Bash call | Command allowlisting |
| `PreToolUse:Edit` | Before Edit call | File protection |
| `PostToolUse:Write` | After Write completes | Auto-format, lint |
| `PostToolUse:Bash` | After Bash completes | Output validation |

### Exit Codes and Environment Variables

| Exit Code | Meaning | Effect |
|-----------|---------|--------|
| `0` | Success | Proceed; stdout shown to Claude as info |
| `1` | Error / Block | Operation blocked; stderr shown as error |
| `2` | Success + Replace | Proceed; stdout replaces original tool result |

**Environment variables available**: `CLAUDE_TOOL_NAME`, `CLAUDE_TOOL_INPUT` (JSON), `CLAUDE_TOOL_OUTPUT` (JSON, PostToolUse only), `CLAUDE_SESSION_ID`, `CLAUDE_PROJECT_DIR`.

### Configuration

```json
{
  "hooks": {
    "PostToolUse:Write": { "command": "python3 .claude/hooks/post-write.py", "timeout": 10000 },
    "PreToolUse:Bash": { "command": "python3 .claude/hooks/pre-bash.py", "timeout": 5000 }
  }
}
```

### Example: Auto-Lint After File Writes

**`.claude/hooks/post-write.py`**:
```python
#!/usr/bin/env python3
"""PostToolUse hook: runs ESLint after .ts/.tsx/.js/.jsx file writes."""
import json, os, subprocess, sys

def main():
    tool_input = json.loads(os.environ.get("CLAUDE_TOOL_INPUT", "{}"))
    file_path = tool_input.get("file_path", "")

    if os.path.splitext(file_path)[1] not in {".ts", ".tsx", ".js", ".jsx"}:
        sys.exit(0)

    result = subprocess.run(
        ["npx", "eslint", "--fix", "--no-error-on-unmatched-pattern", file_path],
        capture_output=True, text=True, timeout=10,
    )
    if result.returncode != 0:
        print(f"ESLint errors in {file_path}:", file=sys.stderr)
        print(result.stdout + result.stderr, file=sys.stderr)
        sys.exit(1)  # Block + show error
    if result.stdout.strip():
        print(f"ESLint auto-fixed issues in {file_path}")
        sys.exit(2)  # Replace tool result with fixed content
    sys.exit(0)

if __name__ == "__main__":
    main()
```

### Example: Block Dangerous Bash Operations

**`.claude/hooks/pre-bash.py`**:
```python
#!/usr/bin/env python3
"""PreToolUse hook: blocks dangerous bash commands."""
import json, os, re, sys

BLOCKED_PATTERNS = [
    r"\brm\s+-rf\s+/",         r"\bgit\s+push\s+--force",
    r"\bgit\s+reset\s+--hard", r"\bdrop\s+database\b",
    r"\bsudo\s+rm\b",          r"\bchmod\s+-R\s+777\b",
    r"\bcurl\b.*\|\s*bash",
]

def main():
    command = json.loads(os.environ.get("CLAUDE_TOOL_INPUT", "{}")).get("command", "")
    for pattern in BLOCKED_PATTERNS:
        if re.search(pattern, command, re.IGNORECASE):
            print(f"BLOCKED: matches dangerous pattern: {pattern}", file=sys.stderr)
            sys.exit(1)
    sys.exit(0)

if __name__ == "__main__":
    main()
```

### The "Never Use LLMs as Linters" Principle

Hooks add latency to every tool invocation. Keep them fast and deterministic:

- **DO**: ESLint, Prettier, mypy, ruff, shellcheck, path allowlists, JSON/YAML schema validation, regex naming checks
- **DON'T**: LLM code review, expensive test suites, external API calls, anything over a few seconds

**Target latency**: Under 1 second per hook.

---

## MCP Server Configuration

MCP servers extend Claude Code with specialized tools. Configure in `.claude/settings.json`:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@anthropic/mcp-playwright"],
      "env": { "DISPLAY": ":0" }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": { "GITHUB_PERSONAL_ACCESS_TOKEN": "${GITHUB_TOKEN}" }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/allowed/dir"]
    },
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    }
  }
}
```

### Browser Debugging: Choosing the Right Approach

| Approach | Best For | Setup | Capabilities |
|----------|----------|-------|-------------|
| **Playwright MCP** | E2E testing, automation | Medium | Full browser control, screenshots, selectors |
| **Chrome DevTools MCP** | Debugging live pages | Low | Console, network tab, DOM inspection |
| **Claude-in-Chrome** | Quick page analysis | Minimal | Page content reading, visual analysis |

**Playwright MCP**: E2E tests, browser automation, screenshots, visual regression. Use when you need programmatic browser control.

**Chrome DevTools MCP**: JS errors, network inspection, performance analysis. Use when debugging a running page.

**Claude-in-Chrome Extension**: Page content summaries, documentation reading, ad-hoc questions. Use when you just need to understand what is on screen.

---

## Git Worktrees for Parallel Development

Git worktrees check out multiple branches in separate directories. Each gets its own Claude Code session.

### Setup and Structure

```bash
cd /path/to/project
git worktree add ../project-feature-auth feature/auth
git worktree add ../project-feature-api feature/api-v2
git worktree add ../project-bugfix-perf bugfix/performance
```

```
workspace/
  project/                  # Main worktree (main branch)
  project-feature-auth/     # Auth feature worktree
  project-feature-api/      # API v2 worktree
  project-bugfix-perf/      # Perf bugfix worktree
```

### Parallel Sessions

Run a separate `claude` in each worktree's terminal. Each session has independent context -- no cross-contamination between features.

### Best Practices

- **Shared config**: Worktrees share `.claude/` from the repo; branch-specific instructions go in that branch's CLAUDE.md
- **Merge strategy**: Complete features independently, merge to main one at a time
- **Cleanup**: `git worktree remove ../project-feature-auth` after merge
- **Sprint pattern**: Create worktrees per sprint item, run `/rpi:research` in parallel across terminals, then plan and implement independently

---

## Voice Prompting

Voice input changes interaction patterns: faster for intent, less precise for technical specifics.

### Best Practices

- **Voice for intent, text for specifics**: "Add a caching layer to the user service" (voice), then specific config in text
- **Voice for research phase**: "Research how the payment pipeline works and identify all currency conversion points"
- **Text for plan review**: Reading and editing plans is better done visually
- **Dictate commit messages**: Voice works well for describing what changed and why

### Voice-Friendly vs Text-Friendly Patterns

```
# Good for voice (intent-heavy):
"Think harder about whether we should use Redis or Memcached for session storage"
"Research all places we make external HTTP calls and list which have retry logic"

# Better as text (precision-heavy):
"Add a Redis cache with TTL of 3600s to getUserById in src/services/user.ts using ioredis"
```

### Combining Voice and Text

1. Voice: "I want to refactor the notification system to support multiple channels"
2. Claude asks clarifying questions
3. Text: provide specific file paths, interfaces, constraints
4. Voice: "That looks good, go ahead and implement it"

---

## Status Line and Background Tasks

### Configuring statusLine

```json
{
  "statusLine": {
    "enabled": true,
    "command": "echo \"branch: $(git branch --show-current) | node: $(node -v) | $(date +%H:%M)\""
  }
}
```

Shows persistent info: current branch, runtime versions, environment indicators, time.

### Background Tasks

The `run_in_background` parameter on the Bash tool runs commands asynchronously. Claude is notified on completion.

```
"Run the full test suite in the background and notify me when it completes"
```

### Monitoring Pattern

Combine statusLine with a hook that updates a status file:

```json
{ "statusLine": { "enabled": true, "command": "echo \"$(git branch --show-current) | tests: $(cat .test-status 2>/dev/null || echo idle)\"" } }
```

---

## Production Configuration Reference

A complete `.claude/` directory for a production project:

```
.claude/
  settings.json          # Hooks, MCP servers, statusLine
  commands/
    rpi:research.md      # Research phase
    rpi:plan.md          # Planning phase
    rpi:implement.md     # Implementation phase
    review.md            # Code review
    test.md              # Run tests with context
  skills/
    api-patterns.md      # API endpoint conventions
    db-migrations.md     # Database migration procedures
    error-handling.md    # Error handling conventions
  agents/
    senior-engineer.md   # Deep research and review
    test-writer.md       # Comprehensive test generation
  hooks/
    post-write.py        # Auto-lint after writes
    pre-bash.py          # Block dangerous commands
    pre-write.py         # Protect critical files
```

**Complete `.claude/settings.json`**:
```json
{
  "hooks": {
    "PostToolUse:Write": { "command": "python3 .claude/hooks/post-write.py", "timeout": 10000 },
    "PreToolUse:Bash": { "command": "python3 .claude/hooks/pre-bash.py", "timeout": 5000 }
  },
  "mcpServers": {
    "sequential-thinking": { "command": "npx", "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"] }
  },
  "statusLine": { "enabled": true, "command": "echo \"$(git branch --show-current) | $(node -v)\"" }
}
```

---

## References

- See [`docs/tools/claude-code-guide.md`](claude-code-guide.md) for basics: CLAUDE.md hierarchy, context management, getting started
- See [`docs/tools/claude-code-settings-reference.md`](claude-code-settings-reference.md) for comprehensive settings documentation
- See [`docs/tools/claude-code-monorepo.md`](claude-code-monorepo.md) for monorepo-specific patterns
- See [`templates/claude/`](../../templates/claude/) for ready-to-use CLAUDE.md and command templates
- [Anthropic: Claude Code Documentation](https://docs.anthropic.com/en/docs/claude-code)
- [Anthropic: Effective Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Anthropic: Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
