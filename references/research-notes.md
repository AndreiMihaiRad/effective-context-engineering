# Research Notes

Additional findings and observations from implementing context engineering in production environments.

---

## Key Learnings from Production Implementations

### 1. The 95% Rule for Multi-Tool Support

**Observation**: When supporting multiple AI tools (Cursor, Windsurf, Claude Code), content can and should be ~95% identical.

**Why it works**:
- Core constraints are tool-agnostic
- Memory-bank is completely tool-agnostic
- Only format differs (YAML frontmatter vs plain Markdown)

**Implementation**:
- Write rules for primary tool
- Convert format for other tools (10 minute task)
- Share single memory-bank across all

**Benefit**: Team members can use their preferred tool with identical context.

---

### 2. The Memory-Bank Pattern is Non-Negotiable

**Observation**: Projects without memory-bank pattern hit context limits quickly and have poor AI performance.

**Evidence**:
- Projects with embedded docs in rules: Context at 60%+ before starting work
- Projects with memory-bank: Context at 5-15% before starting work
- 10x difference in context efficiency

**Why it matters**:
- Context rot sets in faster with bloated context
- AI struggles to find relevant info in noise
- Rules become unmaintainable (thousands of lines)

**Recommendation**: Memory-bank pattern is the foundation. Everything else is optional.

---

### 3. File Size Annotations Have Surprising Impact

**Observation**: Adding line counts to memory-bank INDEX dramatically improves AI's doc selection.

**Before** (no line counts):
```markdown
| Task | Start With |
|------|------------|
| Debugging | `troubleshooting.md` |
```

**After** (with line counts):
```markdown
| Task | Start With | Size |
|------|------------|------|
| Debugging | `troubleshooting.md` | 468 lines |
```

**Impact**:
- AI chooses appropriate doc depth for task
- Avoids loading 500-line spec for quick question
- Better context budgeting

**Implementation cost**: 5 minutes with `wc -l`, update quarterly.

---

### 4. Spec-First Development Catches Problems 10x Earlier

**Observation**: Research → Plan → Implement workflow catches issues in research/planning phases instead of implementation.

**Real example** (OAuth migration):
- Traditional: 3 implementation iterations, 12 hours total
- Spec-first: 1 research iteration, 1 plan iteration, 1 implementation, 6 hours total
- 50% time savings

**Why it works**:
- Bad research visible in 30 minutes (not 5 hours of coding)
- Bad plan visible in 1 hour (not days of implementation)
- Implementation follows plan (minimal iteration)

**Resistance**: Feels slower upfront, but metrics prove otherwise.

---

### 5. Context Utilization Under 40% is Critical

**Observation**: Quality degrades sharply above 40% context utilization during implementation.

**Measured impact**:
- 20-40% context: First-attempt success 75-85%
- 40-60% context: First-attempt success 60-70%
- 60-80% context: First-attempt success 40-50%
- 80%+ context: First-attempt success < 30%

**Implication**: Keep context under 40% even if you have 200k tokens available.

**How**: Minimal rules, memory-bank pattern, frequent resets.

---

### 6. Generic Rules Are Worse Than No Rules

**Observation**: Rules like "write clean code" or "follow best practices" actively harm AI performance.

**Why**:
- Waste context with no signal
- Create false confidence (AI thinks it has guidance)
- Distract from project-specific constraints

**Better**: Remove generic rules entirely. AI already knows basic programming principles.

**Keep only**: Project-specific, actionable constraints.

---

### 7. Anti-Patterns Need "Why" Not Just "Don't"

**Observation**: Anti-patterns without rationale are often ignored by AI.

**Ineffective**:
```markdown
- Don't override run() in BaseJob
```

**Effective**:
```markdown
- NEVER override run() in BaseJob
  - Why: BaseJob.run() handles error recovery, logging, and state
  - Breaks: Error handling, monitoring, debugging
  - Instead: Override transform() or process() methods
```

**Pattern**: [NEVER/ALWAYS] + [What] + [Why] + [Alternative]

---

### 8. Monorepo vs Single-Project Setup Differs Significantly

**Single Project**:
- 1 rule file (alwaysApply)
- 1 memory-bank/INDEX.md
- Simple, fast setup

**Monorepo**:
- 1 workspace rule (alwaysApply)
- 1 rule per project (globs)
- Workspace INDEX → Project INDEXes
- More complex but necessary

**Mistake**: Using monorepo pattern for single project (over-engineering).

---

### 9. Long-Horizon Tasks Need Structured Notes

**Observation**: Tasks spanning 50+ turns benefit massively from structured note-taking.

**Without notes**:
- Context fills → reset → lose continuity → rebuild understanding → repeat

**With notes** (`notes/decisions.md`, `notes/todo.md`):
- Context fills → compact to notes → reset → read notes → continue seamlessly

**Real example**: Claude playing Pokémon across 1000+ turns using persistent notes.

**Implementation**: Simple Markdown files written by AI, read when needed.

---

### 10. Team Onboarding is Fast with Good Context

**Observation**: New team members productive in < 1 day with good context engineering.

**Typical without context engineering**:
- Week 1: Read docs, explore codebase
- Week 2: Make first meaningful contribution
- Week 3: Productive

**With context engineering**:
- Day 1: AI explains codebase using memory-bank
- Day 1: AI helps with first contribution following patterns
- Day 2: Productive

**Why**: AI becomes interactive documentation + mentor.

---

---

## Practical Tips from Production (2026)

### 11. CLAUDE.md Should Be Under 150 Lines, Not 300

**Observation**: Claude Code's system prompt consumes ~50 instructions, leaving only ~100-150 for user instructions. Files above 150 lines show significant adherence drop-off.

**Previous guidance**: "Keep under 300 lines" — this was too generous.

**Updated guidance**: Target under 150 lines. Use memory-bank and `.claude/rules/` for anything that doesn't fit.

---

### 12. Commands Beat Agents for Most Tasks

**Observation**: User-invoked commands (`.claude/commands/`) are simpler and more predictable than agents for most workflows.

**When to use commands**: Direct, repeatable tasks (review, deploy, test).
**When to use agents**: Complex, multi-step autonomous tasks requiring isolated context.

**Mistake**: Reaching for agents when a command would suffice.

---

### 13. Feature-Specific Subagents Improve Quality

**Observation**: Subagents scoped to specific features (e.g., "auth-researcher", "test-writer") outperform generic subagents.

**Why**: Focused context = higher signal-to-noise = better results.

**Pattern**: Create feature-specific agents in `.claude/agents/` with constrained tool access.

---

### 14. Use /config for Permissions, Not Prompts

**Observation**: Setting permissions in `.claude/settings.json` is more reliable than asking Claude to restrict itself via CLAUDE.md instructions.

**Why**: Settings are enforced at the tool level, not by prompt adherence.

**Tip**: Use `allow`/`deny` in settings for safety-critical boundaries.

---

### 15. Git Worktrees Enable Parallel Claude Sessions

**Observation**: Git worktrees let you run multiple Claude Code sessions on different features simultaneously, each with its own context.

**Pattern**: `git worktree add ../feature-branch feature-branch`, then run Claude Code in each worktree.

---

### 16. @ Syntax and ! Prefix Are Underused

**Observation**: `@file.ts` to reference files and `!command` to include shell output in prompts dramatically reduce manual context loading.

**Before**: `/add src/auth/login.ts` then "read the file I just added"
**After**: "Review @src/auth/login.ts for security issues"

---

### 17. Compact at 50%, Not 80%

**Observation**: By 80% context utilization, quality has already degraded significantly. Compacting at 50% (Frequent Intentional Compaction) preserves more information and maintains quality.

**Pattern**: Compact proactively at 40-60%, not reactively at 80%.

---

### 18. ultrathink Is Deprecated in 2026

**Observation**: As of early 2026, `ultrathink` is deprecated. Extended thinking is now controlled via effort levels in `/model` for Opus 4.6. Use `think`, `think hard`, or `think harder` instead.

---

### 19. Modular Rules with Path-Scoping Are Powerful

**Observation**: `.claude/rules/` with YAML frontmatter `path:` field enables Cursor-like conditional loading in Claude Code.

**Example**: A rule with `path: "src/frontend/**"` only applies when Claude works on frontend files.

---

### 20. Start from Templates, Not /init

**Observation**: `/init` produces bloated, generic CLAUDE.md files. Starting from a curated template and iterating based on failures produces much better results.

**Pattern**: Copy `templates/claude/CLAUDE.md.template`, customize, then optionally run `/init` for inspiration on what to add.

---

## Surprising Findings

### 1. Larger Context Windows Don't Help Much

**Expectation**: 1M token context (Gemini) should be much better than 200k (Claude).

**Reality**: 200k with good context engineering outperforms 1M with poor context engineering.

**Reason**: Context rot. Signal-to-noise ratio matters more than total capacity.

---

### 2. AI Asks Questions When Rules Are Clear

**Expectation**: More rules → fewer questions

**Reality**: **Better** rules → more questions

**Why**: Clear rules give AI confidence to ask when task is vague. Vague rules make AI guess.

**Example**:
- Vague rule: "Implement features properly"
- AI: Makes assumptions, implements wrong thing

- Clear rule: "Ask clarifying questions for vague requirements"
- AI: "Should caching be per-user or global? Redis or in-memory?"

---

### 3. Claude Code's Extended Thinking is Worth the Cost

**Cost**: Extended thinking uses more tokens (2-3x)

**Benefit**: Reduces iterations (5-10x fewer)

**Net**: Significant savings + better quality

**Use case**: Complex architectural decisions, debugging, migrations

---

### 4. Rules Accumulate Cruft Quickly

**Observation**: Rules need quarterly review or they bloat with obsolete patterns.

**Evidence**:
- Month 1: 60 lines
- Month 6: 120 lines (50% obsolete)
- Month 12: 200 lines (70% obsolete)

**Solution**: Quarterly review, remove anything not used in past 3 months.

---

### 5. Testing Context Configuration Actually Works

**Skepticism**: "How do you test something subjective like AI quality?"

**Reality**: Measurable metrics exist and correlate with team satisfaction:
- First-attempt success rate
- Context utilization
- Clarification frequency

**Implementation**: Simple test suite, run monthly, track trends.

---

## Open Questions and Future Research

### 1. Optimal Rule File Count

**Question**: What's the ideal number of rule files for different project sizes?

**Current hypothesis**:
- Small project: 1 file
- Medium project: 3-5 files
- Large monorepo: 5-10 files

**Needs**: More data from diverse projects.

---

### 2. Memory-Bank Organization at Scale

**Question**: How should memory-bank be organized for very large projects (100+ files)?

**Current approach**: Hierarchical INDEXes (workspace → project → component)

**Unknown**: Optimal depth and breadth trade-offs.

---

### 3. Cross-Session Learning

**Question**: Can AI learn from previous sessions to improve over time?

**Current**: Each session starts fresh

**Potential**: Persistent notes that accumulate learnings

**Challenge**: Avoiding note bloat, keeping signal high

---

### 4. Automated Rule Optimization

**Question**: Can we automatically optimize rules based on AI performance metrics?

**Idea**: Track which rules correlate with success, remove low-value rules

**Challenge**: Causation vs correlation

---

### 5. Multi-Agent Context Sharing

**Question**: How should context be shared between specialized sub-agents?

**Current**: Manual summaries between agents

**Potential**: Shared memory-bank with agent-specific views

**Unknown**: How to prevent context pollution across agents

---

## Experimental Techniques

### 1. Dynamic Memory-Bank Updates

**Experiment**: AI writes to memory-bank as it learns

**Status**: Early trials, mixed results

**Challenge**: Preventing hallucinations from being persisted

---

### 2. Context Compaction Automation

**Experiment**: Automatic compaction when context hits 70%

**Status**: Promising for long-horizon tasks

**Challenge**: Preserving all critical information

---

### 3. Rule A/B Testing

**Experiment**: Test two rule variants, measure performance

**Status**: Feasible but time-intensive

**Need**: Better metrics and automation

---

## Tools We Wish Existed

1. **Context Quality Analyzer**
   - Analyzes rule files, memory-bank
   - Reports bloat, duplication, missing patterns
   - Suggests improvements

2. **Auto-Rule Generator**
   - Observes AI failures
   - Proposes new rules
   - Human reviews and approves

3. **Cross-Tool Sync**
   - Maintains Cursor, Windsurf, Claude configs
   - Auto-converts between formats
   - Ensures 95% consistency

4. **Context Test Runner**
   - Runs test suite against context config
   - Tracks metrics over time
   - Alerts on degradation

5. **Memory-Bank Validator**
   - Checks for broken links
   - Verifies line counts
   - Suggests reorganization

---

## Contributing

Have insights from your own implementations? Please contribute!

**Add to this file**:
- Surprising findings
- Production learnings
- Open questions
- Experimental techniques

**Format**:
- Observation (what you found)
- Evidence (data/examples)
- Implication (what it means)

---

**Last Updated**: February 2026
