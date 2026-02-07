---
name: code-review
description: "Performs thorough code review checking security, performance, and project patterns"
allowed-tools:
  - Read
  - Grep
  - Glob
---

# Code Review Skill

## Purpose

Perform a thorough code review when asked to review code changes, PRs, or specific files.

## Instructions

When reviewing code:

1. **Read the files** to understand what changed
2. **Check security** — OWASP Top 10 vulnerabilities
3. **Check performance** — N+1 queries, sync calls in async, missing indexes
4. **Check patterns** — does it follow project architecture (hexagonal)?
5. **Check constraints** — see Critical Constraints in CLAUDE.md

## Review Checklist

- [ ] No SQL string concatenation (use parameterized queries)
- [ ] No sync DB calls in async endpoints
- [ ] Pydantic validation on all inputs
- [ ] Domain logic in `domain/`, not in `adapters/`
- [ ] Tests included (unit + integration)
- [ ] Type hints on all functions

## Output Format

```markdown
## Code Review

### Summary
[Overall assessment: Approve / Request Changes]

### Issues Found
- **[Severity]**: [Description] in `file:line`

### Suggestions
- [Optional improvements]
```
