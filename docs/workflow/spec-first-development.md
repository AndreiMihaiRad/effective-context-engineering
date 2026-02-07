# Spec-First Development with AI Agents

> **Key insight**: The specification is more valuable than the code. A bad specification leads to thousands of bad lines of code.

**Attribution**: Based on Anthropic's context engineering research and production workflows from nexus and data-api projects.

---

## Why Specification Matters More Than Code

### The Hierarchy of Impact

```
Bad Research → Bad Plan → Bad Code
    ↓             ↓           ↓
  1 hour      10 hours   100 hours
  wasted      wasted      wasted
```

**Concrete example**:

**Bad research** (1 hour wasted):
- Misunderstand authentication architecture
- Think it uses OAuth when it uses JWT

**Bad plan** (10 hours wasted):
- Implement OAuth integration (wrong approach)
- Create 500 lines of unnecessary code

**Bad code** (100 hours wasted):
- Tests written for wrong system
- Documentation for wrong patterns
- Team confusion and rework

**Key insight**: **Catch problems in the research phase**, not after implementation.

---

## Three-Phase Workflow

### Overview

```
1. Research Phase   → Understand the system
2. Planning Phase   → Specify every change
3. Implementation   → Write code, maintain context
```

**Time allocation** (example):
- Research: 20% of time
- Planning: 30% of time
- Implementation: 50% of time

**Why this works**: Front-load thinking, reduce iteration.

---

## Phase 1: Research

### Goal

Understand how the system works before changing it.

### Process

**Step 1: Define the research question**

```
Clear: "Research how authentication works in this codebase"

Vague: "Look at the code"
```

**Step 2: Have AI explore systematically**

**Claude Code example**:
```bash
claude /clear

"Research how authentication works in this codebase.

Tasks:
1. Find all authentication-related files
2. Identify the auth pattern (JWT, OAuth, sessions, etc.)
3. Locate where tokens are validated
4. Find where user sessions are managed
5. Document the auth flow

Create a research output file with:
- List of relevant files with line numbers
- Auth pattern summary
- Key functions and their locations
"
```

**Step 3: AI creates research output**

**Example output** (`research-auth.md`):

```markdown
# Authentication Research

## Pattern
JWT tokens with RS256 signing

## Key Files

### Token Generation
- `src/auth/tokens.ts` (lines 23-67)
  - `generateToken(user)` - Creates JWT
  - `refreshToken(token)` - Handles refresh

### Token Validation
- `src/middleware/auth.ts` (lines 12-45)
  - `validateToken(req)` - Middleware
  - Checks signature and expiry

### User Sessions
- `src/auth/sessions.ts` (lines 8-89)
  - Session store using Redis
  - TTL: 7 days

## Auth Flow
1. User logs in → `POST /auth/login`
2. `AuthService.login()` validates credentials
3. `generateToken()` creates JWT
4. Token returned to client
5. Client sends token in `Authorization` header
6. Middleware validates on each request

## Dependencies
- jsonwebtoken library
- Redis for session storage
- bcrypt for password hashing
```

### Output Requirements

**Good research output includes**:
- ✅ File paths with line numbers
- ✅ Key functions and their purposes
- ✅ Dependencies and relationships
- ✅ Data flows
- ✅ Architectural patterns

**Bad research output**:
- ❌ Generic descriptions
- ❌ Pasted code without context
- ❌ Missing file locations

### When to Iterate

**Iterate research if**:
- Key questions unanswered
- Contradictory information
- Missing critical files
- Unclear about patterns

**Stop research when**:
- Clear understanding of system
- All relevant files identified
- Patterns documented
- Ready to plan changes

---

## Phase 2: Planning

### Goal

Specify every single change before writing code.

### Process

**Step 1: Create detailed implementation plan**

**Cursor/Windsurf example**:
```
[New chat - fresh context]

"Based on research-auth.md, create an implementation plan for adding OAuth support.

Requirements:
1. List every file to modify
2. Specify what changes in each file
3. Include test cases needed
4. Describe verification steps

Format as a structured plan document."
```

**Claude Code example**:
```bash
claude /clear

"Read research-auth.md and create implementation plan for adding OAuth support.

Include:
- Files to create (with purpose)
- Files to modify (with specific changes)
- Code snippets showing key changes
- Test cases to add
- Migration strategy for existing JWT users
- Verification checklist

Use extended thinking to consider edge cases."
```

**Step 2: AI creates implementation plan**

**Example output** (`plan-oauth.md`):

```markdown
# OAuth Integration Implementation Plan

## Summary
Add OAuth 2.0 support while maintaining JWT for existing users.

## Files to Create

### 1. `src/auth/oauth.ts` (new)
**Purpose**: OAuth provider integration

```typescript
export class OAuthProvider {
  constructor(config: OAuthConfig) { }
  
  async initiateAuth(provider: string): Promise<string> {
    // Returns authorization URL
  }
  
  async handleCallback(code: string): Promise<User> {
    // Exchanges code for token, fetches user
  }
}
```

### 2. `src/config/oauth.ts` (new)
**Purpose**: OAuth configuration

```typescript
export interface OAuthConfig {
  google: {
    clientId: string;
    clientSecret: string;
    redirectUri: string;
  };
  github: { ... };
}
```

## Files to Modify

### 1. `src/auth/tokens.ts`
**Changes**: Add OAuth token handling

**Line 50** (after `generateToken()`):
```typescript
export async function generateOAuthToken(
  user: User,
  provider: string
): Promise<Token> {
  // Include OAuth provider in token claims
  const claims = {
    ...user,
    authMethod: 'oauth',
    provider,
  };
  return generateToken(claims);
}
```

### 2. `src/routes/auth.ts`
**Changes**: Add OAuth routes

**Line 30** (after `/login` route):
```typescript
// OAuth initiation
router.get('/oauth/:provider', async (req, res) => {
  const url = await oauthProvider.initiateAuth(req.params.provider);
  res.redirect(url);
});

// OAuth callback
router.get('/oauth/callback', async (req, res) => {
  const user = await oauthProvider.handleCallback(req.query.code);
  const token = await generateOAuthToken(user, req.query.state);
  res.json({ token });
});
```

## Test Cases

### Unit Tests
1. `tests/auth/oauth.test.ts`
   - `OAuthProvider.initiateAuth()` returns valid URL
   - `OAuthProvider.handleCallback()` exchanges code
   - Token generation includes provider claim

### Integration Tests
2. `tests/integration/oauth.test.ts`
   - Full OAuth flow (mock provider)
   - Verify token contains OAuth claims
   - Existing JWT auth still works

## Migration Strategy

1. Deploy OAuth support (no breaking changes)
2. Announce OAuth availability
3. Gradual user migration
4. JWT remains supported indefinitely

## Verification Checklist

- [ ] OAuth routes accessible
- [ ] Google OAuth flow works end-to-end
- [ ] GitHub OAuth flow works end-to-end
- [ ] Existing JWT auth still functional
- [ ] OAuth tokens validate correctly
- [ ] Tests pass (unit + integration)
- [ ] Documentation updated

## Edge Cases Considered

- User exists with email, tries OAuth (link accounts)
- OAuth fails mid-flow (show error, don't crash)
- Invalid OAuth state parameter (reject)
- OAuth provider is down (graceful degradation)
```

### Plan Quality Criteria

**Good plan**:
- ✅ Specific files and line numbers
- ✅ Code snippets showing changes
- ✅ Test cases defined
- ✅ Edge cases considered
- ✅ Verification steps included

**Bad plan**:
- ❌ "Add OAuth to auth system"
- ❌ No specific files
- ❌ No code examples
- ❌ Missing test strategy

---

## Mandated Intentional Human Review

### Why Human Review Matters

AI can write code faster than humans can review it. **Review the plan, not the code.**

**Why this works**:
- 200-line plan → 20 minutes to review
- 2,000-line PR → 3 hours to review
- Problems caught early → less rework

### What to Review

**In Research Phase**:
- [ ] Is the system understanding correct?
- [ ] Are all relevant files identified?
- [ ] Is the pattern identified correctly?
- [ ] Are there missing pieces?

**In Planning Phase**:
- [ ] Does the plan address the requirement?
- [ ] Are edge cases considered?
- [ ] Is the approach appropriate?
- [ ] Are breaking changes minimized?
- [ ] Is the migration strategy sound?

**In Implementation Phase**:
- [ ] Does code match the plan?
- [ ] Were there plan deviations? Why?
- [ ] Are tests comprehensive?
- [ ] Is documentation updated?

### Review Workflow

**After Research**:
```
1. AI creates research-X.md
2. Human reviews (15 minutes)
3. Human approves or requests changes
4. If approved → proceed to planning
```

**After Planning**:
```
1. AI creates plan-X.md
2. Human reviews (30 minutes)
3. Human approves or requests changes
4. If approved → proceed to implementation
```

**After Implementation**:
```
1. AI implements per plan
2. Human reviews code against plan (30 minutes)
3. If deviations, understand why
4. Approve or request changes
```

---

## Phase 3: Implementation

### Goal

Write code following the plan, maintain context under 40%.

### Process

**Step 1: Execute plan**

**Cursor/Windsurf example**:
```
[New chat - fresh context]

"Implement OAuth support following plan-oauth.md.

Maintain context under 40%.
Create files and make changes as specified.
Write tests as defined in plan.
"
```

**Claude Code example**:
```bash
claude /clear

"Read plan-oauth.md and implement OAuth support.

Keep context utilization under 40%.
Follow the plan exactly.
After creating each file, update plan-oauth.md with [DONE] markers.
"
```

**Step 2: Update plan as you go**

Mark completed items:
```markdown
## Files to Create

### 1. `src/auth/oauth.ts` [DONE]
### 2. `src/config/oauth.ts` [DONE]

## Files to Modify

### 1. `src/auth/tokens.ts` [IN PROGRESS]
### 2. `src/routes/auth.ts` [TODO]
```

**Step 3: Monitor context**

```bash
# Claude Code
claude /cost

# Cursor/Windsurf
[Check token indicator]

Target: Stay under 40%
```

**Step 4: Verify against plan**

After implementation:
```markdown
## Verification Checklist

- [x] OAuth routes accessible
- [x] Google OAuth flow works
- [x] GitHub OAuth flow works
- [x] Existing JWT auth functional
- [x] OAuth tokens validate
- [x] Tests pass
- [x] Documentation updated
```

### Context Management During Implementation

**When context > 40%**:
1. **Finish current file**
2. **Drop irrelevant files** (if using `/drop`)
3. **Or reset and continue** with plan

**Claude Code example**:
```bash
# Context at 45%, finish current task
[Complete current file]

# Reset
claude /clear

# Continue
"Read plan-oauth.md.
Continue implementation from step [X].
Files completed: [list]
Next: [next task]
"
```

---

## Real-World Example: Data API Endpoint

### Research Phase (15 minutes)

**Prompt**:
```
"Research how endpoints are created in this codebase.
Focus on Trino endpoints.
Create research output file."
```

**Output** (`research-endpoint-creation.md`):
```markdown
# Endpoint Creation Research

## Pattern
Declarative TOML + Jinja2 SQL templates

## Key Files
- TOML definition: `definitions/casino/monthly-revenue.toml`
- SQL template: `definitions/casino/monthly-revenue.sql.jinja2`
- Validation: `components/definitions/src/validator.py`

## Flow
1. Define endpoint in TOML (parameters, backend, ACLs)
2. Write SQL template with Jinja2 syntax
3. Validate with `definitions validate`
4. Deploy via Buildkite → S3 → Kong
```

### Planning Phase (30 minutes)

**Prompt**:
```
"Based on research-endpoint-creation.md, create plan for daily-revenue endpoint.

Requirements:
- Parameters: start_date, end_date
- Group by product and region
- Cache for 1 hour
"
```

**Output** (`plan-daily-revenue.md`):
```markdown
# Daily Revenue Endpoint Plan

## Files to Create

### 1. `definitions/casino/daily-revenue.toml`
```toml
[endpoint]
path = "/casino/daily-revenue"
backend = "trino"
cache_ttl = 3600

[[parameters]]
name = "start_date"
type = "date"
required = true

[[parameters]]
name = "end_date"
type = "date"
required = true
```

### 2. `definitions/casino/daily-revenue.sql.jinja2`
```sql
SELECT
  date,
  product,
  region,
  SUM(revenue) as total_revenue
FROM casino.revenue
WHERE date BETWEEN {{ start_date }} AND {{ end_date }}
GROUP BY date, product, region
ORDER BY date DESC, total_revenue DESC
```

## Validation
```bash
definitions validate --strict
definitions render daily-revenue --params '{"start_date": "2025-01-01", "end_date": "2025-01-31"}'
```

## Tests
- Integration test in `tests/integration/casino/test_daily_revenue.py`
- Performance test (should return in < 2s for 30-day range)

## Verification
- [ ] TOML validates
- [ ] SQL renders correctly
- [ ] Test in dev environment
- [ ] Performance acceptable
```

**Human review**: Approved (5 minutes)

### Implementation Phase (15 minutes)

**Prompt**:
```
"Implement daily-revenue endpoint following plan-daily-revenue.md"
```

**Result**:
- Files created as specified
- Validation passes
- Tests written and passing

**Total time**: 65 minutes (15 research + 30 planning + 5 review + 15 implementation)

**Compare to no-plan approach**:
- Jump to coding: 10 minutes
- Realize wrong pattern: 20 minutes wasted
- Rewrite: 30 minutes
- Debug: 20 minutes
- **Total**: 80 minutes + frustration

---

## Benefits of Spec-First Development

### 1. Catch Problems Early

**Research phase** catches:
- Wrong understanding of system
- Missing information
- Wrong patterns

**Planning phase** catches:
- Wrong approach
- Missing edge cases
- Breaking changes

**Implementation phase** catches:
- Code bugs (only)

### 2. Better Communication

**Plan as communication**:
- Team sees what will change
- Early feedback opportunity
- Architectural alignment

### 3. Reduced Context Usage

**Plan serves as memory**:
- Reset context, read plan, continue
- Plan is specification
- Less accumulated context

### 4. Easier Review

**Review 200-line plan vs 2,000-line PR**:
- Plan: 20 minutes
- PR: 3 hours

### 5. Mental Alignment

**Team stays aligned**:
- Research reviewed by team
- Plan reviewed by team
- Implementation follows plan
- No surprises

---

## Summary Workflow

```
┌─────────────┐
│  Research   │ ← Understand system
│   Phase     │   Output: research-X.md
└──────┬──────┘
       │
   [Human Review: 15 min]
       │
       ↓
┌─────────────┐
│  Planning   │ ← Specify changes
│   Phase     │   Output: plan-X.md
└──────┬──────┘
       │
   [Human Review: 30 min]
       │
       ↓
┌─────────────┐
│Implementa-  │ ← Write code
│  tion Phase │   Output: Working code
└──────┬──────┘
       │
   [Human Review: 30 min]
       │
       ↓
     [Done]
```

---

---

## Implementable RPI Templates

Ready-to-use Research-Plan-Implement commands for Claude Code are available in `templates/claude/rpi/`:

- **`/rpi:research [topic]`** — launches research phase, creates structured research output
- **`/rpi:plan [topic]`** — takes research output, creates detailed implementation plan
- **`/rpi:implement [topic]`** — executes plan with context management (compacts at 50%)

### Setup

```bash
# Copy RPI commands to your project
cp -r templates/claude/rpi/commands/rpi .claude/commands/

# Optionally add the senior engineer agent
cp templates/claude/rpi/agents/senior-engineer.md.template .claude/agents/senior-engineer.md
```

See `templates/claude/rpi/README.md` for detailed setup instructions.

---

## Human Leverage Hierarchy

Where human review has the most impact:

| Review Phase | Leverage | Time to Review | Cost of Missing Issues |
|--------------|----------|----------------|----------------------|
| **Research review** | Highest (10x) | 15 minutes | Wrong understanding → wrong everything |
| **Plan review** | High (5x) | 30 minutes | Wrong approach → wasted implementation |
| **Code review** | Standard (1x) | 30 minutes | Bugs → fixes |

**Implication**: Invest MORE review time in research and plans, LESS in code. A 15-minute research review prevents 10 hours of wasted implementation.

---

## References

- See `templates/claude/rpi/` for implementable RPI workflow templates
- See `docs/fundamentals/context-engineering.md` for context principles
- See `docs/best-practices/context-budgeting.md` for context management during implementation
- See `docs/workflow/long-horizon-tasks.md` for extended workflows
- See `examples/` for real-world spec-first examples
