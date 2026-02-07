# Context Budgeting: Managing Context Limits

> **Key principle**: Context is a finite, precious resource. Budget it like you budget memory in performance-critical code.

**Attribution**: Based on production experience, Anthropic research, and real-world context management patterns.

---

## Understanding Context Window Limits

### Context Windows by Model (2025)

| Model | Context Window | Effective Limit | Recommended Usage |
|-------|----------------|-----------------|-------------------|
| **Claude Opus 4.5** | 200k tokens | ~160k tokens | 40% during implementation |
| **Claude Sonnet 4.5** | 200k tokens | ~160k tokens | 40% during implementation |
| **GPT-5.1 Thinking** | 400k tokens | ~320k tokens | 30-40% during implementation |
| **GPT-5.1 Instant** | 400k tokens | ~320k tokens | 30-40% during implementation |
| **Gemini 3 Pro** | 1M tokens | ~800k tokens | 30-40% during implementation |

**Key insights**:
- Larger context ≠ better performance
- Context rot sets in even within limits
- Keep utilization under 40% for best results

### Token Counting Basics

**Rough estimates**:
- 1 token ≈ 0.75 English words
- 100 lines of code ≈ 300-500 tokens
- 1 page of documentation ≈ 500-750 tokens

**Tools**:
- Claude Code: `/cost` command
- Cursor: Token usage in UI
- Windsurf: Token usage in UI
- Manual: [OpenAI Tokenizer](https://platform.openai.com/tokenizer)

---

## The Context Rot Phenomenon

### What Is Context Rot?

**Definition**: As context windows expand, model accuracy degrades—even within stated limits.

**Why it happens**:
1. **Attention Scarcity**: Models have finite "attention budgets"
2. **The n² Problem**: Every token must attend to every other token
3. **Signal Dilution**: Important info gets lost in noise

### Practical Implications

```
Context Quality = (Signal Tokens) / (Total Tokens)
```

**Example**:

**Bad** (low signal-to-noise):
```
Total: 150,000 tokens
Signal: 5,000 tokens (relevant to current task)
Noise: 145,000 tokens (old files, irrelevant docs)
Quality: 3.3%
```

**Good** (high signal-to-noise):
```
Total: 20,000 tokens
Signal: 15,000 tokens (relevant to current task)
Noise: 5,000 tokens (rules, current files)
Quality: 75%
```

**Result**: 20k high-quality context outperforms 150k low-quality context.

---

## File Size Annotations Strategy

### Why Annotate File Sizes?

Helps AI budget context by understanding cost of reading each doc.

### The 4-Tier System

From nexus and data-api production implementations:

**Level 0: Index Files** (< 60 lines)
- **Purpose**: Navigation only
- **Token cost**: ~200 tokens
- **When to read**: Always start here
- **Examples**: `memory-bank/INDEX.md`

**Level 1: Quick References** (< 150 lines)
- **Purpose**: Copy-paste solutions
- **Token cost**: ~500 tokens
- **When to read**: Need syntax/example
- **Examples**: `quick-refs/common_transformations.md`

**Level 2: Component Overviews** (150-350 lines)
- **Purpose**: Understanding architecture
- **Token cost**: ~1,000-1,500 tokens
- **When to read**: Need to understand how something works
- **Examples**: `components/kafka-reader.md`

**Level 3: Deep References** (350+ lines)
- **Purpose**: Comprehensive specs
- **Token cost**: ~1,500-3,000+ tokens
- **When to read**: Complex implementation or troubleshooting
- **Examples**: `workflows/troubleshooting.md` (468 lines)

### Implementation in INDEX.md

```markdown
## File Size Reference

### Lightweight (< 150 lines) - Quick reads
**Token cost**: ~500 tokens each
**When to use**: Quick lookups, syntax references

| File | Lines | Topic |
|------|-------|-------|
| `architecture/overview.md` | 87 | System architecture |
| `components/auth.md` | 112 | Authentication patterns |

### Medium (150-350 lines) - Focused deep-dives
**Token cost**: ~1,000-1,500 tokens each
**When to use**: Understanding components, workflows

| File | Lines | Topic |
|------|-------|-------|
| `workflows/adding-endpoint.md` | 365 | Endpoint creation guide |
| `components/trino-facade.md` | 263 | Trino query execution |

### Large (350-500 lines) - Comprehensive references
**Token cost**: ~1,500-2,000 tokens each
**When to use**: Complex workflows, troubleshooting

| File | Lines | Topic |
|------|-------|-------|
| `workflows/troubleshooting.md` | 468 | Debugging guide |

### Very Large (> 500 lines) - Detailed specifications
**Token cost**: 2,000+ tokens each
**When to use**: Implementation details, complete specs
**Caution**: Only read when necessary

| File | Lines | Topic |
|------|-------|-------|
| `tech-stack/jinja-templating.md` | 460 | SQL template syntax |
| `tech-stack/python-fastapi.md` | 525 | FastAPI patterns |
```

---

## Progressive Disclosure Approach

### The Principle

Load information in layers: overview → details → specifics

### Example Flow

**Task**: "Implement retry logic for Kafka reader"

**Bad approach** (load everything):
```
1. Read all Kafka documentation (2,000 lines = 6,000 tokens)
2. Read all error handling docs (1,500 lines = 4,500 tokens)
3. Read all retry examples (800 lines = 2,400 tokens)
→ Total: 12,900 tokens, mostly irrelevant
```

**Good approach** (progressive disclosure):
```
1. Read INDEX.md (56 lines = 200 tokens)
   → Points to components/kafka-readers.md

2. Read components/kafka-readers.md (263 lines = 800 tokens)
   → Notes retry pattern in JdbcReader lines 45-67

3. Read JdbcReader lines 45-67 (23 lines = 70 tokens)
   → Understand pattern

4. Implement following pattern
→ Total: 1,070 tokens, all highly relevant
```

**Result**: 12x fewer tokens, 10x higher signal.

---

## When to Compact vs Reset

### Context Reset (Preferred)

**When**:
- Finishing one task, starting unrelated task
- Context > 40% before starting new work
- Response quality degrading

**How**:
```bash
# Cursor
[Start new chat]

# Windsurf
[New conversation]

# Claude Code
claude /clear
```

**Benefits**:
- Fresh, clean context
- No accumulated cruft
- Consistent quality

### Compaction (When Reset Not Possible)

**When**:
- Must maintain continuity (long task)
- Research → Plan → Implement flow
- Context approaching limits mid-task

**How** (Claude Code example):
```
"Create compaction file with:
- All modified file paths
- Key architectural decisions made
- Remaining tasks
- Current implementation state

Maximize recall first, then refine for precision."
```

**Save output, then**:
```bash
claude /clear
# Read compaction file and continue
```

---

## Context Utilization Metrics

### Target Utilization: 40-60% with Frequent Compaction

**Nuanced guidance**:
- Below 40% is ideal for complex tasks requiring deep exploration
- 40-60% is the practical working range — use Frequent Intentional Compaction (FIC) here
- Above 60% triggers compaction urgently — quality degrades noticeably
- Never let context exceed 80% — by this point, adherence has already dropped significantly

### How to Monitor

**Cursor**:
- Check token indicator in UI
- Aim for < 40% of available context

**Windsurf**:
- Check token indicator in UI
- Monitor during long sessions

**Claude Code**:
```bash
claude /cost
```
Output shows current utilization.

### What to Do at Different Levels

**< 20% utilization**: Ideal
- Continue normally
- Can load additional docs freely

**20-40% utilization**: Healthy
- Good working range for complex tasks
- Monitor as you add files

**40-60% utilization**: Active management needed
- Use Frequent Intentional Compaction (FIC)
- Consider dropping irrelevant files
- Compact proactively, don't wait

**60-80% utilization**: Critical
- Quality already degrading noticeably
- Compact or reset immediately
- Finish current subtask then reset

**> 80% utilization**: Danger zone
- Responses will degrade significantly
- Context rot is severe
- Reset immediately — compaction at this point preserves less information

---

## Tools to Monitor Context Usage

### Built-in Tools

**Cursor**:
- Token counter in bottom status bar
- Shows current / total tokens
- Updates in real-time

**Windsurf**:
- Token counter in UI
- Context usage percentage
- Updates during session

**Claude Code**:
- `/cost` command
- Shows:
  - Input tokens
  - Output tokens
  - Total cost
  - Context utilization

### Manual Monitoring

**Check loaded files**:
```bash
# Claude Code
/files
```

**Estimate token count**:
```
Rule of thumb:
- 100 lines of code ≈ 400 tokens
- Rules files (avg) ≈ 300 tokens
- Index file ≈ 200 tokens
- Component doc (medium) ≈ 1,000 tokens
```

**Budget example** (Claude Sonnet, 200k window):
```
Target: 40% = 80,000 tokens

Current usage:
- Rules (3 files): 900 tokens
- INDEX.md: 200 tokens
- 2 component docs: 2,000 tokens
- Current file: 500 tokens
- Conversation history: 3,000 tokens
─────────────────────────────
Total: 6,600 tokens (3.3%)

Remaining budget: 73,400 tokens
→ Plenty of room for exploration
```

---

## Real-World Examples

### Example 1: Data API Platform

**Total memory-bank**: 21 files, ~6,049 lines, ~18,000 tokens

**Typical task**: "Create new endpoint"

**Context budget**:
```
Rules: 300 tokens
INDEX.md: 200 tokens
adding-endpoint.md: 1,100 tokens (365 lines)
Current files: 1,000 tokens
Conversation: 2,000 tokens
─────────────────────────────
Total: 4,600 tokens (2.3% of 200k)
```

**Result**: 97.7% context available for implementation.

### Example 2: Nexus Monorepo

**Total memory-bank**: 45+ files, ~20,000 lines, ~60,000 tokens

**Typical task**: "Add new transformation"

**Context budget**:
```
Rules (2 files): 600 tokens
Workspace INDEX: 200 tokens
Project INDEX: 500 tokens
transformations_overview.md: 1,200 tokens
Conversation: 3,000 tokens
Current files: 2,000 tokens
─────────────────────────────
Total: 7,500 tokens (3.75% of 200k)
```

**Result**: 96.25% context available.

### Example 3: Bad Context Management

**Anti-pattern**: Load everything upfront

```
All rules (10 files): 5,000 tokens
All memory-bank (pre-loaded): 60,000 tokens
Current files: 5,000 tokens
Conversation: 10,000 tokens
─────────────────────────────
Total: 80,000 tokens (40% BEFORE starting task!)
```

**Result**: Hit 40% before doing anything, context rot imminent.

---

## Best Practices Summary

### 1. Keep Rules Minimal

```
Target: < 100 lines per rule
Token cost: ~300 tokens per rule
Maximum: 3-5 rule files
```

### 2. Use File Size Annotations

```markdown
| File | Lines | Topic |
|------|-------|-------|
| `overview.md` | 87 | Architecture |
```

### 3. Progressive Disclosure

```
1. INDEX (200 tokens)
2. Overview (800 tokens)
3. Specific section (300 tokens)
```

### 4. Monitor Utilization

```
Target: < 40% during implementation
Warning: 40-60%
Critical: > 60%
```

### 5. Reset Between Tasks

```bash
# Finish task 1
[reset]
# Start task 2 (fresh context)
```

### 6. Compact When Necessary

```
Long task approaching limit:
→ Compact to progress file
→ Reset with compaction context
→ Continue
```

---

## Context Budgeting Checklist

Before starting work:

- [ ] Rules are < 100 lines each
- [ ] INDEX.md has file size annotations
- [ ] Current context < 20% (fresh session)
- [ ] Relevant docs identified but not pre-loaded
- [ ] Plan to reset between unrelated tasks

During work:

- [ ] Monitor context utilization
- [ ] Load docs progressively (not all at once)
- [ ] Drop irrelevant files when done
- [ ] Context stays < 40%

When context grows:

- [ ] At 40%: Consider what to drop
- [ ] At 60%: Finish subtask, then reset
- [ ] At 80%: Compact or reset immediately

---

## References

- [Effective Context Engineering](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) - Anthropic research on context limits
- See `docs/fundamentals/context-engineering.md` for underlying principles
- See `docs/fundamentals/memory-bank-pattern.md` for the memory-bank pattern
- See `docs/best-practices/anti-patterns.md` for what to avoid
- See `examples/` for real-world context budgets
