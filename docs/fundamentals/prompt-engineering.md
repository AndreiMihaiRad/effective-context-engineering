# Effective Task Prompts for AI Agents

> **Key insight**: With good context management in place (memory-bank, rules, CLAUDE.md), your prompts can be remarkably simple. The AI already knows your project.

**Attribution**: This document synthesizes patterns from production projects (nexus, data-api) and the ai-presentation reference.

---

## The Optimal Task Prompt

### Core Structure

```
[Action]: [Target]

Context: [Only if not in rules - 1-2 lines max]

Requirements:
- Specific requirement 1
- Specific requirement 2

Constraints:
- Constraint 1 (if any)
```

### Examples by Task Type

**Bug Fix**
```
Fix: Login fails silently when API returns 401

The error handler in auth.ts catches the error but doesn't surface it to the user.
Show a toast notification on auth failures.
```

**New Feature**
```
Implement: Retry logic for Kafka reader

Requirements:
- 3 retries with exponential backoff
- Log each retry attempt
- Follow the pattern in JdbcReader

Constraint: Don't modify the KafkaReaderFactory interface.
```

**Refactoring**
```
Refactor: Extract state management from IngestScript into separate class

Goal: Reduce IngestScript from 400 to <200 lines.
Keep the same public interface.
```

**Code Review**
```
Review: PR #234 for the new transformation

Focus on:
- Performance (this runs on 10M+ rows)
- Error handling edge cases
- Test coverage
```

---

## Effective Patterns

### 1. Be Specific, Not Verbose

```
❌ "Please help me fix the authentication issue we've been having
    with the login system. It seems like something is wrong..."

✅ "Fix: Auth token not refreshing in useAuth hook"
```

### 2. One Task at a Time

```
❌ "Fix the login bug, add logout functionality, and update the tests"

✅ "Fix: Login bug in auth.ts"
   [wait for completion]
   "Add: Logout functionality following login pattern"
   [wait for completion]
   "Update: Tests for auth changes"
```

### 3. Reference Existing Patterns

```
❌ "Create a new reader that connects to the API, handles errors,
    retries on failure, logs everything..."

✅ "Create: RestApiReader following the pattern in JdbcReader"
```

### 4. State Constraints Upfront

```
❌ "Add caching to the API"
   [AI implements Redis]
   "Actually, we can't use Redis, only in-memory"

✅ "Add: In-memory caching to API responses
    Constraint: No external dependencies (Redis, etc.)"
```

### 5. Use Thinking Keywords for Complex Tasks

For Claude Code and other tools with extended thinking modes:

```
"ultrathink about the architecture for this new feature"

"think hard about edge cases in this error handling"
```

---

## Anti-Patterns

### Generic Requests

```
❌ "Make it better"
❌ "Clean up this code"
❌ "Follow best practices"

✅ "Reduce nesting to max 3 levels"
✅ "Extract the validation logic to a separate function"
✅ "Add error handling for network failures"
```

### Over-Specifying the Obvious

```
❌ "Create a function that takes parameters, has a return type,
    includes proper TypeScript types, handles errors..."

✅ "Create: validateUserInput function for the registration form"
```
The AI knows TypeScript conventions—your rules already specify them.

### Assuming Memory

```
❌ "Use the same approach as before"
❌ "Remember what we discussed about auth"

✅ "Follow the JWT pattern in auth/tokens.ts"
✅ "Use refresh token rotation as in useAuth hook"
```
Each session starts fresh. Reference code, not conversations.

### Multiple Unrelated Tasks

```
❌ "Fix the login bug, update the README, and check why tests are slow"

✅ Focus on one task. Start a new conversation for unrelated work.
```

---

## Quick Reference

### Common Task Prompts

| Task | Prompt Template |
|------|-----------------|
| **Bug fix** | `Fix: [symptom] in [file/component]` |
| **Feature** | `Implement: [feature] following [pattern]` |
| **Refactor** | `Refactor: [target] to [goal]` |
| **Review** | `Review: [code/PR] for [focus areas]` |
| **Explain** | `Explain: How [component] handles [scenario]` |
| **Test** | `Add tests for: [component] covering [cases]` |

### Thinking Keywords (Claude Code)

| Keyword | Use When |
|---------|----------|
| `think` | Moderate complexity |
| `think hard` | Important decisions |
| `ultrathink` | Architecture, complex debugging |

### Context Clearing

Start fresh between unrelated tasks:
- **Cursor**: New chat
- **Windsurf**: New conversation
- **Claude Code**: `/clear`

---

## Real-World Examples

### From nexus (Spark Library)

**Good**:
```
Implement: KafkaAvroReader following the pattern in KafkaJsonReader

Requirements:
- Support Confluent Schema Registry
- Handle schema evolution gracefully
- Add tests covering schema compatibility

Reference: fd-nexus/src/fd_nexus/kafka/readers/kafka_json_reader.py
```

**Why it works**:
- Clear action and target
- References existing pattern (AI can read and follow it)
- Specific requirements
- Points to concrete code location

**Bad**:
```
We need to read Avro data from Kafka. The data uses schemas from the Confluent
Schema Registry and we need to handle schema evolution properly. Make sure
it works with all the different schema compatibility modes and has good error
handling and logging like our other readers.
```

**Why it's bad**:
- Too verbose (the reference pattern already has error handling and logging)
- No concrete reference point
- Over-specifies what's already in the patterns

### From data-api (FastAPI Platform)

**Good**:
```
Create: New Trino endpoint for daily casino revenue

TOML: definitions/casino/daily-revenue.toml
SQL: definitions/casino/daily-revenue.sql.jinja2

Requirements:
- Date range parameters (start_date, end_date)
- Group by product and region
- Cache for 1 hour

Reference: definitions/casino/monthly-revenue.toml for similar pattern
```

**Why it works**:
- Specific files to create
- Clear requirements
- References similar existing endpoint
- Includes caching requirement upfront

**Bad**:
```
Add a new endpoint for casino revenue data
```

**Why it's bad**:
- Vague (daily? monthly? weekly?)
- No parameters specified
- No caching guidance
- No reference to existing patterns

---

## Summary

| Principle | Practice |
|-----------|----------|
| **Simple prompts** | Context does the heavy lifting |
| **Specific > verbose** | "Fix X in Y" not paragraphs |
| **One task** | Clear scope, better results |
| **Reference patterns** | "Follow UserService" not specifications |
| **Constraints upfront** | Saves back-and-forth |

---

## References

- See `docs/fundamentals/context-engineering.md` for the foundation
- See `docs/tools/` for tool-specific prompt guidance
- See `examples/` for real-world prompt examples
