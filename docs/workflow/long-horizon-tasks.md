# Long-Horizon Tasks: Advanced Context Management

> **Key insight**: For tasks spanning hours or days, standard context management breaks down. Special techniques are required.

**Attribution**: Based on Anthropic research (Claude playing Pokémon), production experience, and long-running migration tasks.

---

## When Standard Context Management Breaks Down

### What Are Long-Horizon Tasks?

**Definition**: Tasks requiring:
- Many turns (50+ conversation exchanges)
- Extensive exploration (many files, deep investigation)
- Cumulative state (decisions build on decisions)
- Time span of hours to days

**Examples**:
- Large codebase migrations
- Complex architectural refactoring
- System-wide debugging
- Multi-component feature implementations
- Research-heavy explorations

### Why They're Challenging

**Problem 1: Context window limits**
```
Turn 1-20: Explore system
Turn 21-40: Design solution
Turn 41-60: Implement changes
→ Turn 60: Context at 90%, quality degrading
```

**Problem 2: Context pollution**
```
Early exploration loads 50 files
Most are now irrelevant
But still consuming context
New work suffers from noise
```

**Problem 3: Loss of continuity**
```
Must reset context (hits limit)
Loses architectural decisions
Loses reasoning history
Must rebuild understanding
```

---

## Technique 1: Intentional Compaction

### What It Is

Summarizing conversation to preserve critical information while freeing context.

### When to Use

- Context approaching 50% of limit (don't wait until 80% — quality has already degraded by then)
- Task not complete but must continue
- Can't afford to lose current state
- Use Frequent Intentional Compaction (FIC) at 40-60% for best results

### Implementation Strategy

**Step 1: Maximize Recall**

Capture ALL potentially relevant information:
```
"Create compaction file capturing:
- All architectural decisions made
- All files modified (with line numbers)
- All bugs found and fixed
- All pending tasks
- All key learnings
- Current implementation state

Maximize recall - include everything important."
```

**Step 2: Refine for Precision**

Remove noise while keeping signal:
```markdown
# Compaction Output v1 (5,000 words - too verbose)

[AI includes every detail, very verbose]

→ "Refine this compaction for precision.
   Keep all critical info but remove redundancy."

# Compaction Output v2 (1,500 words - good)

[AI keeps decisions, removes verbose explanations]
```

**Step 3: Preserve Anchors**

Critical information to always include:
- **File paths** that were modified
- **Key decisions** and their rationale
- **Error messages** encountered and resolutions
- **Next steps** planned

### Example Compaction

**Task**: Migrate authentication from JWT to OAuth

**Compaction file** (`progress-oauth-migration.md`):

```markdown
# OAuth Migration Progress

## Completed
- [x] Research phase (see research-auth.md)
- [x] Planning phase (see plan-oauth.md)
- [x] OAuth provider implementation
- [x] Token generation updates
- [x] Route modifications

## Files Modified

### Created
1. `src/auth/oauth.ts` (120 lines)
   - OAuthProvider class
   - Google and GitHub integrations
   - Token exchange logic

2. `src/config/oauth.ts` (45 lines)
   - OAuth configuration interface
   - Provider-specific configs

### Modified
1. `src/auth/tokens.ts`
   - Line 50: Added `generateOAuthToken()`
   - Line 89: Added `validateOAuthToken()`

2. `src/routes/auth.ts`
   - Line 30: Added `/oauth/:provider` route
   - Line 45: Added `/oauth/callback` route

3. `src/middleware/auth.ts`
   - Line 23: Updated validation to handle OAuth tokens

## Key Decisions

### Decision 1: Dual Auth Support
**Rationale**: Maintain JWT for existing users, add OAuth for new users
**Impact**: No breaking changes, gradual migration

### Decision 2: Provider Linking
**Approach**: If OAuth email matches existing user, link accounts
**Implementation**: In `OAuthProvider.handleCallback()` lines 67-89

### Decision 3: Token Claims
**Structure**: Include `authMethod` and `provider` in token claims
**Reason**: Distinguish OAuth from JWT in middleware

## Pending Tasks

1. Testing
   - [ ] Unit tests for OAuthProvider
   - [ ] Integration tests for OAuth flow
   - [ ] Verify existing JWT still works

2. Documentation
   - [ ] Update API docs
   - [ ] Add OAuth setup guide
   - [ ] Document account linking

3. Deployment
   - [ ] Configure OAuth providers in prod
   - [ ] Deploy backend changes
   - [ ] Update frontend login UI

## Issues Encountered

### Issue 1: Token Refresh
**Problem**: OAuth tokens don't support refresh (stateless)
**Solution**: Use refresh tokens from OAuth provider
**Status**: Implemented in `oauth.ts` lines 100-115

### Issue 2: Account Linking Edge Case
**Problem**: What if OAuth email is verified but JWT email isn't?
**Solution**: Trust OAuth verification, mark email as verified
**Status**: Implemented in `oauth.ts` line 75

## Next Steps (Priority Order)

1. Write unit tests for `OAuthProvider`
2. Write integration tests for OAuth flow
3. Test account linking scenarios
4. Update API documentation
5. Configure prod OAuth credentials
```

**Usage**:
```bash
# Context at 85%, save progress
[AI creates compaction file]

# Reset
claude /clear

# Continue
"Read progress-oauth-migration.md and continue OAuth migration.
Next task: Write unit tests for OAuthProvider."
```

---

## Technique 2: Structured Note-Taking (Agentic Memory)

### What It Is

Agent writes notes persisted OUTSIDE context window, retrieves when needed.

### When to Use

- Task spans multiple sessions
- Accumulating learnings over time
- Complex exploration with many branches
- Need to maintain state across context resets

### Implementation

**Note Categories**:
```
notes/
├── decisions.md    # Architectural decisions
├── context.md      # System understanding
├── todo.md         # Pending tasks
└── learnings.md    # Key insights
```

**Writing Notes**:
```
"As you work, write notes to notes/decisions.md when making architectural decisions.

Format:
## Decision: [Title]
**Date**: YYYY-MM-DD
**Context**: Why this decision was needed
**Decision**: What was decided
**Alternatives**: What else was considered
**Rationale**: Why this choice
"
```

**Reading Notes**:
```
"Before implementing, read notes/decisions.md to understand previous architectural choices."
```

### Real-World Example: Claude Playing Pokémon

**Task**: Play Pokémon across thousands of turns

**Challenge**: Single context window can't hold entire game history

**Solution**: Structured notes

**Notes maintained**:

**Objectives** (`notes/objectives.md`):
```markdown
## Current Goal
Defeat Elite Four

## Sub-Goals
- [ ] Grind team to level 50+
- [x] Obtain HM Surf
- [x] Teach Surf to Lapras
- [ ] Stock up on Full Restores

## Strategy
Focus on training Charizard and Lapras (type advantages)
```

**Map Knowledge** (`notes/map.md`):
```markdown
## Visited Locations
- Pallet Town (starting town)
- Route 1 (wild Pokémon)
- Viridian City (Pokémon Center)
- Viridian Forest (caught Pikachu)
- Pewter City (defeated Brock)
[...]

## Important NPCs
- Professor Oak (Pallet Town) - Gives Pokédex
- Brock (Pewter Gym) - Rock-type leader
```

**Party State** (`notes/party.md`):
```markdown
## Current Party

1. Charizard (Lv 48)
   - Flamethrower
   - Wing Attack
   - Slash
   - Fire Spin

2. Lapras (Lv 45)
   - Surf
   - Ice Beam
   - Body Slam
   - Confuse Ray
[...]
```

**Usage Pattern**:
```
Turn 1: Reads objectives, determines next goal
Turn 10: Updates party state after evolution
Turn 20: Adds new location to map
Turn 30: Reads objectives, adjusts strategy
[...]
Turn 1000: Still has context via notes
```

### Implementing in Your Project

**Setup**:
```bash
mkdir -p notes
touch notes/{decisions,context,todo,learnings}.md
```

**System Prompt Addition**:
```markdown
## Note-Taking Protocol

### When to Write Notes
- Making architectural decisions → notes/decisions.md
- Learning system behavior → notes/learnings.md
- Adding TODOs → notes/todo.md
- Understanding grows → notes/context.md

### Note Format
Keep concise (2-3 sentences per entry).
Include date and context.

### When to Read Notes
Before starting new phase, read relevant notes.
```

**Example Workflow**:
```bash
# Session 1
"Implement OAuth. Write decisions to notes/decisions.md"
[Work for 1 hour, context approaching limit]

# Session 2 (fresh context)
claude /clear

"Read notes/decisions.md and notes/todo.md.
Continue OAuth implementation from where we left off."
```

---

## Technique 3: Sub-Agent Architecture

### What It Is

Specialized sub-agents handle focused tasks, returning condensed summaries to lead agent.

### When to Use

- Complex exploration needed
- Want to isolate context pollution
- Can decompose task into independent subtasks
- Budget allows multiple LLM calls

### Pattern

```
Lead Agent (Orchestrator)
├── Explorer Sub-Agent
│   - Deep file exploration
│   - Returns: "Found 3 files, pattern is X"
│   - Context: 50k tokens (isolated)
│
├── Analyzer Sub-Agent
│   - Bug analysis
│   - Returns: "Root cause at line 45 in auth.ts"
│   - Context: 30k tokens (isolated)
│
└── Implementer Sub-Agent
    - Code implementation
    - Returns: "Implemented, tests passing"
    - Context: 40k tokens (isolated)

Lead Agent Context: 5k tokens (only summaries)
```

### Implementation

**Lead Agent**:
```
"Break this migration into subtasks:

1. Exploration: Find all authentication-related files
2. Analysis: Determine current auth pattern
3. Design: Plan OAuth integration approach
4. Implementation: Write code

For each subtask, launch a focused sub-agent.
Return summary to me.
"
```

**Sub-Agent 1 (Explorer)**:
```
[New conversation]

"Explore this codebase for authentication files.

Return summary with:
- File paths
- Key functions
- Authentication pattern

Don't return full file contents.
"

→ Returns 500-word summary
```

**Sub-Agent 2 (Analyzer)**:
```
[New conversation]

"Based on this summary [paste explorer summary], analyze the auth pattern.

Return summary with:
- Pattern type (JWT/OAuth/sessions)
- Strengths and weaknesses
- Integration points for OAuth

Don't repeat exploration details.
"

→ Returns 300-word summary
```

**Lead Agent**:
```
[Has only summaries - 800 words total]

"Based on exploration and analysis summaries, create implementation plan."

→ Uses condensed info, no context pollution
```

### Benefits

**Context Isolation**:
- Each sub-agent's 50k context doesn't affect lead agent
- Lead agent stays lean (< 10k tokens)
- No accumulated pollution

**Focused Expertise**:
- Explorer optimizes for finding files
- Analyzer optimizes for pattern detection
- Implementer optimizes for code generation

**Parallelization**:
- Sub-agents can run in parallel
- Faster overall completion

### Trade-offs

**Costs**:
- More LLM calls = higher cost
- More latency (sequential sub-agents)

**Complexity**:
- Requires good task decomposition
- Summaries must be high-quality
- Coordination overhead

**When It's Worth It**:
- Very complex tasks (> 100 turns)
- Budget allows
- Time to set up is acceptable

---

## Technique 4: Context Reset with Continuity

### When to Use

- Context approaching limits
- Natural break point in work
- Simpler than sub-agents
- Want fresh context but need continuity

### Process

**Step 1: Create handoff document**

```
"Create handoff document for next session:

Include:
- Current state (what's done)
- Pending tasks (what's next)
- Key decisions (why things are this way)
- Important file locations

Keep under 500 lines.
"
```

**Step 2: Reset context**

```bash
claude /clear
```

**Step 3: Load handoff and continue**

```
"Read handoff.md and continue the task.

Next task: [specific next step]
"
```

### Example

**Session 1** (60 turns, context at 50%):
```
"Create handoff.md summarizing OAuth migration progress"

→ handoff.md created (400 lines)
```

**Session 2** (fresh start):
```bash
claude /clear

"Read handoff.md.

Continue OAuth migration.
Next: Implement unit tests for OAuthProvider.
"
```

**Result**: Fresh context, full continuity.

---

## Choosing the Right Technique

| Technique | Use When | Complexity | Cost |
|-----------|----------|------------|------|
| **Compaction** | Context filling, must continue | Medium | Low |
| **Structured Notes** | Multi-session tasks | Medium | Low |
| **Sub-Agents** | Complex exploration, high budget | High | High |
| **Context Reset** | Natural break points | Low | Low |

**Decision Tree**:

```
Task < 50 turns?
└─ Yes → Standard context management
└─ No → Continue

Has natural break points?
└─ Yes → Context Reset with handoff
└─ No → Continue

Budget allows multiple LLM calls?
└─ Yes → Sub-Agent Architecture
└─ No → Continue

Single session or multi-session?
└─ Single → Compaction
└─ Multi → Structured Notes
```

---

## Real-World Example: Large-Scale Migration

**Task**: Migrate 50-file codebase from Redux to Zustand

**Approach**: Structured Notes + Context Reset

**Session 1: Research** (30 turns)
```
- Explore Redux usage patterns
- Document in notes/context.md
- List all files using Redux
- Context at 65% → Reset
```

**Session 2: Planning** (25 turns)
```
- Read notes/context.md
- Create migration plan
- Document decisions in notes/decisions.md
- Context at 55% → Reset
```

**Sessions 3-10: Implementation** (20 turns each)
```
Each session:
1. Read notes/todo.md
2. Pick next component
3. Migrate following plan
4. Update notes/todo.md
5. Reset for next session
```

**Result**: 50 files migrated, context never exceeded 70%.

---

## Summary

### Techniques Recap

**Compaction**:
- Summarize conversation preserving decisions
- Use when hitting context limits
- Maximize recall, then refine precision

**Structured Notes**:
- Persist state outside context
- Use for multi-session tasks
- Categories: decisions, context, todo, learnings

**Sub-Agents**:
- Isolate exploration from main context
- Use for complex, decomposable tasks
- Returns summaries only

**Context Reset**:
- Fresh start with handoff document
- Use at natural break points
- Simple but effective

### Best Practices

1. **Plan for long horizon upfront** - Don't wait for context limits
2. **Document decisions immediately** - Don't rely on memory
3. **Reset liberally** - Fresh context > accumulated context
4. **Use file sizes** - Budget based on doc sizes
5. **Test continuity** - Verify handoff docs work

---

## References

- [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) - Anthropic research
- See `docs/fundamentals/context-engineering.md` for core principles
- See `docs/best-practices/context-budgeting.md` for context management
- See `docs/workflow/spec-first-development.md` for structured workflows
