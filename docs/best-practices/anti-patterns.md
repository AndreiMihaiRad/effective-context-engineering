# Context Engineering Anti-Patterns

> **Key insight**: Most context engineering problems come from violating simple principles. Learn what NOT to do.

**Attribution**: Lessons from real-world failures and production debugging.

---

## The Kitchen Sink Approach

### Anti-Pattern: Rules That Are Too Long

**The Problem**:
```
.cursorrules (2,500 lines)
├── Project overview (200 lines)
├── Complete architecture docs (500 lines)
├── All API endpoints (400 lines)
├── All database schemas (300 lines)
├── All transformations (400 lines)
├── Testing guide (200 lines)
├── Deployment guide (200 lines)
└── Troubleshooting (300 lines)
```

**Why it fails**:
- Bloats context window for every session
- Most content irrelevant for any given task
- Hard to maintain (edit 2,500 lines?)
- Duplicate content across tools

**The Fix**:

```
.cursor/rules/workspace.mdc (44 lines)
├── Overview
├── Memory-bank navigation
└── Core standards

memory-bank/ (external docs)
├── INDEX.md (navigation)
├── architecture/
├── components/
└── workflows/
```

**Result**: Rules 44 lines, docs available on-demand.

---

## Generic Advice That Doesn't Constrain

### Anti-Pattern: "Write Clean Code"

**The Problem**:
```markdown
# Rules

- Write clean code
- Follow best practices
- Be consistent
- Think about performance
- Handle errors properly
- Write good tests
- Use meaningful names
- Keep functions small
```

**Why it fails**:
- Too vague to be actionable
- Could apply to any project (not specific)
- Doesn't guide behavior
- AI already knows these generalities

**The Fix**:
```markdown
# Rules

## Naming Constraints
- Functions: `snake_case` (e.g., `fetch_user_data`)
- Classes: `PascalCase` (e.g., `UserRepository`)
- Constants: `UPPER_CASE` (e.g., `MAX_RETRIES`)

## Function Constraints
- Max 50 lines per function
- Max 3 parameters (use config objects for more)
- Single responsibility (refactor if doing multiple things)

## Error Handling
- Custom exceptions in `exceptions.py`
- Never catch generic `Exception` (be specific)
- Always log errors to Datadog with context

## Testing
- pytest fixtures in `conftest.py`
- 80%+ coverage for new code
- Integration tests in `tests/integration/`
```

**Result**: Specific, actionable, project-specific constraints.

---

## Pre-Loading Everything

### Anti-Pattern: Embedding All Documentation in Rules

**The Problem**:
```yaml
# api-docs.mdc (1,800 lines)

## All API Endpoints

### GET /users
**Description**: Fetches all users
**Parameters**: ...
**Response**: ...
**Example**: ...
[50 lines of documentation]

### POST /users
[50 lines of documentation]

### GET /posts
[50 lines of documentation]

[... 30 more endpoints, 1,500+ lines]
```

**Why it fails**:
- Loads 1,800 lines for every session
- 95% irrelevant for any single task
- Context window bloated immediately
- Hard to update (find specific endpoint in 1,800 lines)

**The Fix**:
```yaml
# api.mdc (60 lines)

## API Constraints
- FastAPI with dependency injection
- Pydantic models for request/response
- Async handlers for I/O operations

## Anti-Patterns
- No business logic in route handlers
- No direct database access (use repositories)

## Memory-Bank
See `memory-bank/api/INDEX.md` for:
- Endpoint catalog
- Implementation patterns
- Testing strategies
```

```markdown
# memory-bank/api/INDEX.md

## Navigation by Task
| Task | Start With | Size |
|------|------------|------|
| Creating endpoint | `workflows/adding-endpoint.md` | 365 lines |
| Endpoint reference | `components/endpoints-reference.md` | 890 lines |
```

**Result**: Rules 60 lines, docs retrieved when needed.

---

## Duplicating Content Across Files

### Anti-Pattern: Same Information Everywhere

**The Problem**:
```
.cursor/rules/workspace.mdc
→ Lists all 50 transformations

.cursor/rules/transformations.mdc
→ Lists all 50 transformations again

CLAUDE.md
→ Lists all 50 transformations again

memory-bank/transformations.md
→ Lists all 50 transformations again (detailed)

README.md
→ Lists top 10 transformations
```

**Why it fails**:
- Updates require changing 5 files
- Versions drift (inconsistency)
- Bloats context unnecessarily
- Maintenance nightmare

**The Fix**:

**Single Source of Truth**:
```
memory-bank/transformations/registry.md
→ Complete list of 50 transformations (authoritative)
```

**References**:
```yaml
# workspace.mdc
50+ transformations in `fd_nexus/spark/transforms/registry.py`
See `memory-bank/transformations/` for details

# CLAUDE.md
## Transformations
50+ pre-built transformations available.
See `memory-bank/transformations/registry.md` for catalog.

# README.md
For transformation catalog, see `memory-bank/transformations/registry.md`
```

**Result**: One source of truth, references everywhere else.

---

## Over-Specifying the Obvious

### Anti-Pattern: Explaining What AI Already Knows

**The Problem**:
```markdown
## Creating a TypeScript Function

When creating a new function in TypeScript:
1. Use the `function` keyword or arrow function syntax
2. Specify parameter types
3. Specify return type
4. Use camelCase for function names
5. Add JSDoc comments
6. Handle potential errors
7. Write unit tests
8. Export if needed for other modules

Example:
```typescript
/**
 * Fetches user by ID
 * @param id User ID
 * @returns User object
 */
export async function fetchUser(id: string): Promise<User> {
  try {
    const response = await fetch(`/api/users/${id}`);
    return await response.json();
  } catch (error) {
    console.error('Failed to fetch user:', error);
    throw error;
  }
}
```
```

**Why it fails**:
- AI already knows TypeScript syntax
- Wastes context on basics
- Doesn't add project-specific value

**The Fix**:
```markdown
## Function Patterns

- Use Zod for runtime validation
- React Query for data fetching (not raw `fetch`)
- Custom hooks in `hooks/api/` directory

Example:
```typescript
// Good: Use React Query
export function useUser(id: string) {
  return useQuery({
    queryKey: ['user', id],
    queryFn: () => api.users.get(id),
    // Zod validates response
  });
}

// Bad: Direct fetch calls
export async function fetchUser(id: string) {
  const res = await fetch(`/api/users/${id}`);
  return res.json(); // No validation!
}
```
```

**Result**: Project-specific patterns, not language basics.

---

## Assuming Memory Across Sessions

### Anti-Pattern: Referencing Previous Conversations

**The Problem**:

**Day 1**:
```
Human: "Let's use Redis for caching"
AI: [Implements Redis caching]
```

**Day 5** (new session):
```
Human: "Add caching to the new endpoint like we discussed"
AI: [Confused - no memory of Day 1]
```

**Why it fails**:
- Each session starts fresh
- No conversation memory
- AI can't "remember what we discussed"

**The Fix**:

**Document Decisions**:
```markdown
# .cursor/rules/caching.mdc

## Caching Strategy
- Use Redis for distributed caching
- TTL: 1 hour for user data, 5 minutes for real-time data
- Cache keys: `namespace:entity:id` (e.g., `users:profile:123`)

## Implementation
See `memory-bank/caching/redis-patterns.md` for:
- Connection setup
- Cache-aside pattern
- Invalidation strategies
```

**Prompt**:
```
"Add caching to the new endpoint following the Redis pattern in caching.mdc"
```

**Result**: AI has context from rules, not conversation history.

---

## Multiple Unrelated Tasks in One Prompt

### Anti-Pattern: "Do Everything at Once"

**The Problem**:
```
"Fix the login bug, add logout functionality, update the tests,
refactor the authentication module, add password reset,
and update the API documentation"
```

**Why it fails**:
- Context jumps between unrelated concerns
- Hard to track what's done
- Increases error risk
- Difficult to review changes
- Context pollution

**The Fix**:

**Task 1** (new chat):
```
"Fix: Login fails silently when API returns 401"
[Complete, review, commit]
```

**Task 2** (new chat):
```
"Implement: Logout functionality following login pattern"
[Complete, review, commit]
```

**Task 3** (new chat):
```
"Add: Tests for authentication changes"
[Complete, review, commit]
```

**Result**: Focused context, clear scope, easier review.

---

## Not Clearing Context Between Tasks

### Anti-Pattern: Accumulated Context Bloat

**The Problem**:

**Turn 1**: "Debug database connection issue"
→ Loads database code, config, logs (10,000 tokens)

**Turn 15**: [Issue fixed]

**Turn 16**: "Add new API endpoint"
→ Context still has all database code from earlier
→ Irrelevant to API endpoint task
→ Context at 60%+ before even starting

**Why it fails**:
- Old context pollutes new tasks
- Context budget wasted on irrelevant info
- Lower quality responses (context rot)

**The Fix**:

**After completing a task**:
```bash
# Cursor: Start new chat
# Windsurf: New conversation
# Claude Code: /clear
```

**Fresh context for new task**:
```
Turn 1: "Add new API endpoint"
→ Loads API code, patterns (3,000 tokens)
→ Context starts clean
→ Better quality responses
```

**Result**: Fresh, focused context for each task.

---

## Ignoring File Size Annotations

### Anti-Pattern: No Context Budgeting

**The Problem**:
```markdown
# memory-bank/INDEX.md

## Documentation

- architecture.md
- components.md
- workflows.md
- troubleshooting.md
[... no line counts]
```

**Why it fails**:
- AI doesn't know cost of reading each doc
- May load massive file unnecessarily
- No way to choose appropriate depth

**The Fix**:
```markdown
# memory-bank/INDEX.md

## File Size Reference

### Lightweight (< 150 lines) - Quick reads
| File | Lines | Topic |
|------|-------|-------|
| `architecture/overview.md` | 87 | System overview |

### Medium (150-350 lines) - Focused deep-dives
| File | Lines | Topic |
|------|-------|-------|
| `workflows/adding-endpoint.md` | 365 | Endpoint creation |

### Large (> 350 lines) - Comprehensive references
| File | Lines | Topic |
|------|-------|-------|
| `workflows/troubleshooting.md` | 468 | Debugging guide |
```

**Result**: AI can choose appropriate doc based on context budget.

---

## Tool-Specific Anti-Patterns

### Cursor: Overly Broad Globs

**The Problem**:
```yaml
---
globs: ["**/*"]  # Matches everything!
---
```

**Why it fails**:
- Defeats purpose of conditional loading
- Rule loads for every file
- Might as well use `alwaysApply`

**The Fix**:
```yaml
---
globs: ["src/api/**/*"]  # Specific to API code
---
```

### Windsurf: Too Many Rule Files

**The Problem**:
```
.windsurf/rules/
├── workspace.md
├── frontend.md
├── backend.md
├── api.md
├── database.md
├── auth.md
├── testing.md
├── deployment.md
└── [10+ files]
```

**Why it fails**:
- ALL rules load for EVERY session in Windsurf
- Context bloated before you start
- No conditional loading

**The Fix**:
```
.windsurf/rules/
├── workspace.md (40 lines)
├── frontend.md (70 lines)
└── backend.md (65 lines)

memory-bank/ (details)
```

### Claude Code: Not Using /clear

**The Problem**:
```
# Single 200-turn conversation
Turn 1-50: Feature A
Turn 51-100: Feature B
Turn 101-150: Feature C
Turn 151-200: Feature D
→ Context at 95%, responses degrading
```

**Why it fails**:
- Accumulated context from all features
- Context rot sets in
- Lower quality responses

**The Fix**:
```
# Feature A
Turns 1-50 → claude /clear

# Feature B (fresh context)
Turns 1-40 → claude /clear

# Feature C (fresh context)
Turns 1-35 → claude /clear
```

---

## Using LLMs as Linters

### Anti-Pattern: AI-Powered Validation in Hooks

**The Problem**:
```json
{
  "hooks": {
    "PostToolUse": [{
      "matcher": "Write|Edit",
      "command": "claude --prompt 'Review this file for issues' $CLAUDE_FILE_PATH"
    }]
  }
}
```

**Why it fails**:
- Hooks run on EVERY tool use — LLM calls are too slow
- Non-deterministic — same file may pass or fail randomly
- Expensive — each hook call costs API tokens
- Recursive — LLM calls in hooks can trigger more hooks

**The Fix**:
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

**Rule**: Hooks must be fast, deterministic, and lightweight. Use traditional tools (ESLint, Ruff, Prettier), not AI.

---

## Auto-Generating CLAUDE.md with /init

### Anti-Pattern: Trusting /init Output As-Is

**The Problem**:
```bash
claude /init
# Produces 400+ lines of generic, verbose content
# User commits it without editing
```

**Why it fails**:
- `/init` produces bloated, generic content
- Often exceeds 150-line limit significantly
- Includes obvious information the AI already knows
- Misses project-specific constraints that matter most

**The Fix**:
```bash
# Start from curated template
cp templates/claude/CLAUDE.md.template CLAUDE.md

# Optionally run /init for inspiration
claude /init > /tmp/init-output.md

# Cherry-pick useful sections, keep under 150 lines
```

**Result**: Curated, concise CLAUDE.md that adds real value.

---

## Summary: Anti-Patterns Checklist

Avoid these common mistakes:

- [ ] ❌ Rules over 100 lines (move to memory-bank)
- [ ] ❌ Generic advice ("write clean code")
- [ ] ❌ Pre-loading all documentation
- [ ] ❌ Duplicating content across files
- [ ] ❌ Over-specifying obvious concepts
- [ ] ❌ Assuming memory across sessions
- [ ] ❌ Multiple unrelated tasks in one prompt
- [ ] ❌ Not clearing context between tasks
- [ ] ❌ No file size annotations
- [ ] ❌ Overly broad glob patterns (Cursor)
- [ ] ❌ Too many rule files (Windsurf)
- [ ] ❌ Not using /clear (Claude Code)
- [ ] ❌ Using LLMs as linters in hooks (use ESLint/Ruff instead)
- [ ] ❌ Auto-generating CLAUDE.md with /init (use templates instead)

---

## References

- See `docs/best-practices/rule-writing.md` for what TO do
- See `docs/best-practices/context-budgeting.md` for sizing guidance
- See `docs/fundamentals/context-engineering.md` for underlying principles
- See `examples/` for real-world good examples
