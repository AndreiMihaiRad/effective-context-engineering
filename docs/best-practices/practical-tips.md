# Practical Tips from Production

> **Key insight**: These tips come from real-world production use of Claude Code, Cursor, and Windsurf. They're organized by category for quick reference.

**Attribution**: Synthesized from production implementations, community best practices, and the claude-code-best-practices repository.

---

## Workflow Tips

### 1. Research Before You Plan, Plan Before You Code

```
Never: "Implement OAuth support"
Always: "Research auth patterns → Create plan → Implement per plan"
```

Front-loading research saves 10x vs. discovering problems during implementation.

### 2. Compact at 50%, Not 80%

By 80% context utilization, quality has already degraded. Use **Frequent Intentional Compaction (FIC)** at 40-60%:

```
"Compact our progress. Include: completed tasks, decisions, remaining work, files modified."
```

Then `/clear` and continue with the compacted summary.

### 3. One Task Per Session

```
❌ Single 200-turn conversation covering 5 features
✅ /clear between each feature
```

Fresh context produces consistently better results than accumulated context.

### 4. Use the Human Leverage Hierarchy

Where human review has the most impact:
1. **Research reviews** (10x leverage) — catch wrong assumptions early
2. **Plan reviews** (5x leverage) — catch wrong approaches
3. **Code reviews** (1x leverage) — catch bugs

Spend more time reviewing research and plans than code.

---

## Tooling Tips

### 5. Start from Templates, Not /init

`/init` produces bloated, generic content. Start from `templates/claude/CLAUDE.md.template` and iterate based on failures.

### 6. Use @ Syntax for File References

```
"Review @src/auth/login.ts for security issues"
"Compare @src/old.ts with @src/new.ts"
```

Much faster than `/add` + "read the file".

### 7. Use ! Prefix for Shell Output

```
"Fix the failing tests shown in !npm test"
"The error is in !cat logs/error.log"
```

Includes command output directly in context.

### 8. Configure Permissions in Settings

Set `allow`/`deny` in `.claude/settings.json` for safety-critical boundaries. This is more reliable than prompt-based restrictions.

```json
{
  "permissions": {
    "allow": ["Bash(npm test:*)", "Bash(git diff:*)"],
    "deny": ["Bash(rm -rf:*)", "Bash(git push --force:*)"]
  }
}
```

### 9. Commands > Agents for Most Tasks

User-invoked commands (`/review`, `/test`) are simpler and more predictable than agents. Use agents only for complex, multi-step autonomous tasks.

### 10. Git Worktrees for Parallel Sessions

```bash
git worktree add ../feature-branch feature-branch
# Run Claude Code in each worktree independently
```

Each worktree gets its own context window.

---

## Debugging Tips

### 11. Read Before You Fix

```
❌ "Fix the bug in auth"
✅ "Read @src/auth/login.ts and @src/auth/tokens.ts, then explain the authentication flow before suggesting fixes"
```

Understanding first, fixing second.

### 12. Use Extended Thinking for Complex Bugs

```
"think harder about why the authentication intermittently fails"
```

Invest thinking budget upfront for complex debugging.

### 13. Isolate with Sub-Agents

For complex debugging, use a sub-agent to investigate without polluting your main context:

```
"Launch a sub-agent to research the authentication failure. Return a summary of root cause and fix."
```

---

## Prompt Tips

### 14. Place Critical Instructions at Beginning AND End

Claude pays most attention to the start and end of CLAUDE.md. Put your most important constraints in both positions.

### 15. Use WHY/WHAT/HOW Framework

Structure CLAUDE.md as:
1. **WHY** — Purpose, critical constraints
2. **WHAT** — Stack, architecture, structure
3. **HOW** — Commands, standards, navigation

### 16. Be Specific, Not Generic

```
❌ "Write clean code"
✅ "Use Pydantic for all input validation. Never catch generic Exception."
```

If a rule could apply to any project, it's too generic.

### 17. Include Anti-Pattern Rationale

```
❌ "Don't override run()"
✅ "NEVER override run() in BaseJob — Why: breaks error recovery, logging, and state management. Instead: override transform() or process()."
```

Anti-patterns with rationale are followed more reliably.

### 18. Keep CLAUDE.md Under 150 Lines

Claude Code's system prompt consumes ~50 instructions, leaving ~100-150 for yours. Adherence drops significantly above 150 lines.

---

## Organization Tips

### 19. Separate Constraints from Documentation

- **Rules** (CLAUDE.md, .claude/rules/) = WHAT to do and NOT do
- **Memory-bank** = HOW things work in detail

### 20. Use File Size Annotations

```markdown
| File | Lines | Topic |
|------|-------|-------|
| `architecture.md` | 87 | System design |
| `troubleshooting.md` | 468 | Debugging guide |
```

AI uses line counts to budget context appropriately.

---

## References

- See `references/research-notes.md` for detailed production learnings
- See `docs/best-practices/anti-patterns.md` for what to avoid
- See `docs/best-practices/context-budgeting.md` for context management
- See `templates/claude/` for ready-to-use templates
