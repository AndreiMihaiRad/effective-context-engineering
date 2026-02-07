# Rule Writing Best Practices

> **Core principle**: Start minimal, iterate based on failures. Rules should constrain behavior, not document it.

**Attribution**: Lessons learned from production implementations (nexus, data-api) and real-world failures.

---

## The Golden Rules

### 1. Start Minimal, Iterate Based on Failures

**Anti-pattern**:
```
Day 1: Write 500 lines of rules anticipating every edge case
```

**Best practice**:
```
Day 1: Write 50 lines covering core constraints
Day N: Add rule when you observe a failure pattern
```

**Example from nexus**:

**Initial workspace.mdc** (30 lines):
```yaml
---
description: Workspace overview
alwaysApply: true
---

# Workspace

## Projects
- fd-nexus: Spark library
- orbit-utils: Airflow operators

## Memory-Bank
See `memory-bank/INDEX.md`

## Code Standards
- Black-compatible formatting
- Type hints required
```

**After observing failures** (44 lines):
```yaml
# Added after AI kept using wrong Python versions
| Package | Python |
|---------|--------|
| fd-nexus | 3.10+ |
| orbit-utils | 3.12+ |

# Added after AI suggested wrong package manager
## Package Management
Both projects use `hatch`
```

**Key insight**: Each addition addresses an observed failure.

---

### 2. Keep Rules Under 100 Lines Each

**Why 100 lines?**
- Readable in 2 minutes
- Forces focus on essentials
- Leaves room for multiple rules

**How to stay under 100**:
- Split by concern, not by size
- Move details to memory-bank
- Focus on constraints, not documentation

**Example**:

❌ **Single 400-line rule file**:
```
# Mega-rules.md (400 lines)

## Architecture (100 lines)
## API Patterns (100 lines)
## Database (100 lines)
## Testing (100 lines)
```

✅ **Four focused rule files**:
```
workspace.md (50 lines)     → Overview, standards
api.md (70 lines)           → API constraints
database.md (60 lines)      → DB patterns, anti-patterns
testing.md (40 lines)       → Test requirements
```

---

### 3. Split by Concern, Not by Size

**Bad split** (arbitrary size):
```
rules-part1.md (100 lines)
rules-part2.md (100 lines)
rules-part3.md (100 lines)
```

**Good split** (logical concerns):
```
workspace.md (50 lines)      → Workspace overview
frontend.md (80 lines)       → React, TypeScript
backend.md (70 lines)        → Python, FastAPI
infrastructure.md (60 lines) → Docker, AWS
```

**Benefits**:
- Clear responsibility
- Easy to find rules
- Easier to maintain

---

### 4. Specific Over Generic

**Generic (useless)**:
```markdown
- Write clean code
- Follow best practices
- Be consistent
- Think about performance
- Handle errors properly
```

**Specific (actionable)**:
```markdown
- Never override `run()` in BaseJob
- Use Delta Lake for transactional writes
- Check 50+ existing transformations before creating new ones
- Always use parameterized queries (not string concatenation)
- Retry logic: 3 attempts, exponential backoff
```

**Test**: If a rule could apply to any project, it's too generic.

---

### 5. Use Clear Sections with Headers

**Bad**:
```markdown
Project uses React with TypeScript and we use functional components
not class components and hooks are required and we use React Query
for data fetching and state is managed with Zustand...
```

**Good**:
```markdown
## Tech Stack
- React 18
- TypeScript strict mode
- React Query (server state)
- Zustand (client state)

## Component Patterns
- Functional components only
- Hooks required
- No class components

## Data Fetching
- React Query for all API calls
- Custom hooks in `hooks/api/`
```

**Structure**:
- `##` for main sections
- `-` for lists
- Code blocks for examples

---

### 6. Prefer Constraints Over Documentation

**Documentation (belongs in memory-bank)**:
```markdown
## Authentication System

The authentication system uses JWT tokens with RS256 signing.
Tokens are issued by the auth service and validated by the API gateway.
The flow works as follows:
1. User logs in with credentials
2. Auth service validates against database
3. JWT token is generated with user claims
[... 50 more lines ...]
```

**Constraints (belongs in rules)**:
```markdown
## Authentication Constraints

- Use JWT tokens from auth service (don't generate locally)
- Validate tokens in middleware, not route handlers
- Never store tokens in localStorage (use httpOnly cookies)

For implementation details, see `memory-bank/auth/authentication.md`
```

**Rule of thumb**:
- Rules: What/what not to do
- Memory-bank: How it works

---

### 7. Reference Memory-Bank for Details

**Pattern**:
```markdown
## [Topic]

### Quick Constraints
- [3-5 key constraints]

### Memory-Bank
For detailed implementation, see `memory-bank/[topic]/[file].md`
```

**Example**:
```markdown
## API Endpoints

### Constraints
- Use FastAPI dependency injection
- Pydantic models for request/response
- Async handlers for I/O operations

### Memory-Bank
- Creation workflow: `memory-bank/workflows/adding-endpoint.md`
- Patterns: `memory-bank/components/api-patterns.md`
```

---

### 8. Version Control Your Rules

**Commit**:
```bash
.cursor/rules/*.mdc
.windsurf/rules/*.md
CLAUDE.md
memory-bank/
```

**Don't commit** (user-specific):
```bash
~/.cursor/settings.json
~/.claude/CLAUDE.md
```

**Benefits**:
- Team shares same context
- Track rule evolution
- Review rule changes in PRs

---

## Real Examples from Production

### Example 1: nexus workspace.mdc (44 lines)

**What it does well**:

✅ **Minimal** (44 lines)
```yaml
---
alwaysApply: true
---

# Nexus Workspace Rules
```

✅ **Clear project overview**
```markdown
| Package | Purpose | Python |
|---------|---------|--------|
| fd-nexus | Spark library | 3.10+ |
| orbit-utils | Airflow operators | 3.12+ |
```

✅ **Points to memory-bank**
```markdown
## Memory-Bank Navigation
Start with `memory-bank/INDEX.md`
```

✅ **Specific standards**
```markdown
## Code Standards
- Formatting: Black-compatible (ruff for fd-nexus)
- Type annotations: Required for all public APIs
- Package management: hatch
```

### Example 2: nexus fd-nexus.mdc (59 lines)

**What it does well**:

✅ **Clear scope** (via globs)
```yaml
---
globs: ["fd-nexus/**/*"]
---
```

✅ **Actionable patterns**
```markdown
## Key Patterns

### Script Pattern
BaseJob → BaseIngestScript → SpecificIngestScript

### Transformation Registry
50+ pre-built transformations in registry.py
```

✅ **Concrete anti-patterns**
```markdown
## Anti-Patterns
- NEVER override `run()` in BaseJob
- NEVER use argparse directly (use typed config classes)
- ALWAYS check 50+ existing transformations before creating new ones
```

✅ **Memory-bank reference**
```markdown
## Memory-Bank First
**ALWAYS consult** `memory-bank/fd-nexus/INDEX.md`
```

### Example 3: data-api CLAUDE.md (266 lines)

**What it does well**:

✅ **Structured sections**
```markdown
## Workspace Overview (50 lines)
## Development Commands (40 lines)
## Architecture (60 lines)
## Memory-Bank Navigation (30 lines)
## Quick Reference (40 lines)
## Critical Constraints (16 lines)
```

✅ **Actionable commands**
```bash
cd data-api-core
poetry run pytest              # Tests
poetry run uvicorn ...         # Dev server
```

✅ **Security constraints**
```markdown
### Security (DO NOT)
- ❌ String concatenation in SQL templates
- ❌ Overly permissive ACLs
- ✅ Use parameterized queries
```

✅ **Performance constraints**
```markdown
### Performance (DO NOT)
- ❌ Full table scans
- ❌ Functions on filter columns
- ✅ Filter on partition/indexed columns
```

---

## Common Pitfalls and Solutions

### Pitfall 1: Too Many Rules

**Problem**:
```
.cursor/rules/
├── workspace.mdc
├── frontend.mdc
├── backend.mdc
├── api.mdc
├── database.mdc
├── auth.mdc
├── testing.mdc
├── deployment.mdc
├── monitoring.mdc
└── [... 10 more files]
```

**Solution**:
```
.cursor/rules/
├── workspace.mdc    (alwaysApply: true)
├── frontend.mdc     (globs: ["frontend/**/*"])
└── backend.mdc      (globs: ["backend/**/*"])

memory-bank/
├── api/
├── database/
├── auth/
├── testing/
[... rest as documentation]
```

**Guideline**: 3-5 rule files maximum.

### Pitfall 2: Embedding Full Documentation

**Problem**:
```yaml
# api-endpoints.mdc (1,200 lines)

## All Endpoints

### /users
[Full documentation: 100 lines]

### /posts
[Full documentation: 100 lines]

[... 10 more endpoints, 1,000+ lines]
```

**Solution**:
```yaml
# api.mdc (60 lines)

## API Constraints
- FastAPI dependency injection
- Pydantic validation
- Async for I/O

## Memory-Bank
See `memory-bank/api/INDEX.md` for:
- Endpoint catalog
- Implementation patterns
- Testing strategies
```

### Pitfall 3: Vague Guidance

**Problem**:
```markdown
- Write good tests
- Make code performant
- Handle errors gracefully
- Use appropriate design patterns
```

**Solution**:
```markdown
- Tests: pytest fixtures in conftest.py, 80%+ coverage
- Performance: < 100ms p95 for API endpoints
- Errors: Use custom exceptions in `exceptions.py`, log to Datadog
- Patterns: Repository pattern for data access (see `memory-bank/patterns/`)
```

### Pitfall 4: Duplicating Content

**Problem**:
```
workspace.mdc        → Lists all 50 transformations
frontend.mdc         → Documents API endpoints
CLAUDE.md            → Repeats transformation list
memory-bank/api.md   → Repeats endpoints
README.md            → Repeats architecture
```

**Solution**:
```
workspace.mdc        → "50+ transformations in registry.py"
CLAUDE.md            → "See memory-bank/INDEX.md"
memory-bank/api.md   → Single source of truth for endpoints
README.md            → Links to memory-bank
```

**Principle**: Single source of truth, references everywhere else.

### Pitfall 5: Not Updating Rules

**Problem**:
```
# Rules written 6 months ago, never updated
# Code has evolved, rules are now wrong
```

**Solution**:
```
# Review rules when:
- Onboarding new team members (observe confusion points)
- After repeated AI mistakes (add constraint)
- Major architecture changes (update rules)
- Quarterly rule review (remove obsolete rules)
```

**Practice**: Treat rules like code—they evolve.

---

## Rule Writing Workflow

### Step 1: Start Minimal

```yaml
---
description: Project rules
alwaysApply: true
---

# Project Rules

## Memory-Bank
See `memory-bank/INDEX.md`

## Tech Stack
- [Language and version]
- [Framework]
- [Key libraries]

## Code Standards
- [Formatting tool]
- [Type checking]
```

### Step 2: Observe Failures

Use AI assistant for a week, note when it:
- Makes wrong assumptions
- Violates conventions
- Misses patterns
- Suggests anti-patterns

### Step 3: Add Specific Constraints

For each failure, add a specific rule:

```markdown
## Anti-Patterns

# Added after AI suggested sync DB calls in async endpoint
- Never use sync database calls in async endpoints

# Added after AI created new transformation instead of using existing
- Check 50+ existing transformations before creating new ones
```

### Step 4: Refine and Split

When a rule file approaches 100 lines:
1. Identify logical sections
2. Split into separate files (if using Cursor globs)
3. Or move details to memory-bank

### Step 5: Review Regularly

Monthly review:
- Remove obsolete rules
- Consolidate similar rules
- Update for architecture changes
- Verify memory-bank references are current

---

## Testing Your Rules

### Test 1: Can AI Find Documentation?

**Prompt**: "Where can I find documentation about [topic]?"

**Expected**: References memory-bank INDEX, finds correct doc.

### Test 2: Does AI Follow Constraints?

**Prompt**: "Create a new [component] for [feature]"

**Expected**: Follows patterns and anti-patterns from rules.

### Test 3: Are Rules Too Generic?

**Review**: For each rule, ask "Could this apply to any project?"

- If yes → too generic, make specific or remove
- If no → good, keep it

### Test 4: New Team Member Test

**Process**: Have new team member use AI assistant

**Observe**: What confusions arise? What rules are missing?

**Update**: Add constraints for observed failure patterns.

---

## Summary Checklist

When writing rules, ensure:

- [ ] Each rule file is under 100 lines
- [ ] Rules are split by concern, not arbitrarily
- [ ] Constraints are specific and actionable
- [ ] Generic advice is removed
- [ ] Memory-bank is referenced for details
- [ ] Clear sections with headers
- [ ] Concrete examples where helpful
- [ ] Anti-patterns are explicit
- [ ] Version controlled with project
- [ ] Tested with real tasks

---

## References

- See `docs/fundamentals/memory-bank-pattern.md` for the memory-bank pattern
- See `docs/best-practices/anti-patterns.md` for what to avoid
- See `docs/best-practices/context-budgeting.md` for sizing guidance
- See `templates/` for ready-to-use templates
- See `examples/` for real-world implementations
