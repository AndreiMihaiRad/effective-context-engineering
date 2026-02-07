# Context Engineering: The Foundation of Effective AI Development

> "Context engineering is building dynamic systems to provide the right information and tools in the right format such that the LLM can plausibly accomplish the task."

**Attribution**: This document synthesizes concepts from production implementations in the nexus (Spark pipelines), data-api (FastAPI platform), and ai-presentation projects.

---

## 1. The Evolution: From Prompt Engineering to Context Engineering

### What Changed?

**Prompt Engineering** focused on crafting clever instructions—the art of asking the right question. It treated the LLM as a static interface where better wording yields better results.

**Context Engineering** addresses a broader challenge: managing ALL tokens available to a language model during inference. This includes:
- System instructions
- Available tools
- External data (retrieved docs, code, APIs)
- Message history
- User preferences and memory

### The Core Principle

> "Find the smallest possible set of high-signal tokens that maximize desired outcomes."

This principle represents a fundamental shift: context is not about giving the model MORE information, but about giving it the RIGHT information at the RIGHT time.

### Why Context > Prompts

| Prompt Engineering | Context Engineering |
|-------------------|---------------------|
| Static instructions | Dynamic information assembly |
| Single-shot optimization | System-level orchestration |
| Focused on wording | Focused on information architecture |
| Works within limits | Designs around limits |

Prompt engineering is now considered **a subset of context engineering**—wording still matters, but it's one component of a larger system.

---

## 2. Understanding Context Limitations

### The Context Rot Phenomenon

Research demonstrates a counterintuitive finding: **as context windows expand, model accuracy degrades**. This is called "context rot."

Why does this happen?

1. **Attention Scarcity**: LLMs have finite "attention budgets" similar to human working memory limitations
2. **The n² Problem**: Transformer architecture requires every token to attend to every other token, creating n² pairwise relationships
3. **Signal Dilution**: Important information gets "lost" among less relevant tokens

### Practical Implications

```
Context Quality = (Signal Tokens) / (Total Tokens)
```

Adding more context doesn't help if it dilutes the signal-to-noise ratio. A 200k token context window filled with marginally relevant information performs worse than a 10k token window with precisely curated content.

### Context Windows by Model (2025)

| Model | Context Window | Thinking Mode | Coding Benchmark | Notes |
|-------|---------------|---------------|------------------|-------|
| Claude Opus 4.5 | 200k tokens | Extended thinking | 80.9% SWE-bench | Best long-horizon coding |
| Claude Sonnet 4.5 | 200k tokens | Extended thinking | 77.2% SWE-bench | Balanced performance/cost |
| GPT-5.1 Thinking | 400k tokens | Adaptive (Light→Heavy) | 74.9% SWE-bench | Adjustable thinking effort |
| GPT-5.1 Instant | 400k tokens | None | — | Fast, everyday tasks |
| Gemini 3 Pro | 1M tokens | Deep Think | 50%+ vs 2.5 Pro | Best vibe coding, largest context |

**Key insight**: A larger context window is not inherently better. Claude's 200K window with superior context management often outperforms larger windows with lower attention quality.

---

## 3. System Prompt Engineering

### Finding the Right "Altitude"

System prompts exist on a spectrum:

**Too Brittle (Low Altitude)**
```
When the user asks about authentication, respond with exactly:
"Authentication uses JWT tokens stored in httpOnly cookies..."
```
- Creates fragility
- Requires constant maintenance
- Doesn't handle edge cases

**Too Vague (High Altitude)**
```
Be helpful with coding questions.
```
- Insufficient guidance
- Inconsistent behavior
- Wastes model's reasoning capacity on basics

**Optimal (Right Altitude)**
```
You are a senior data engineer specializing in Spark and Databricks.
When implementing data pipelines:
- Prefer Delta Lake for transactional writes
- Always consider partitioning and clustering strategies
- Reference the transformations registry in fd-nexus
```
- Specific enough to guide behavior
- Flexible enough for model heuristics
- Anchored to project context

### Structuring System Prompts

Use clear delimiters to organize sections:

```markdown
# Role
You are a senior developer working on {project_name}.

# Constraints
- Never modify files in /core without explicit approval
- Always run tests before suggesting commits

# Codebase Context
<architecture>
The application uses a hexagonal architecture with:
- /domain: Business logic (pure functions)
- /adapters: External integrations
- /ports: Interface definitions
</architecture>

# Response Format
- Start with a brief summary
- Include code examples when applicable
- End with potential edge cases to consider
```

### XML Tags vs Markdown

Both work, but be consistent:

| Format | Best For |
|--------|----------|
| XML tags `<section>` | Complex nested structures, Claude models |
| Markdown headers `##` | Human-readable, simpler prompts |
| YAML frontmatter | Metadata and configuration |

---

## 4. Tool Design for Agents

### The Fundamental Rule

> "If engineers cannot definitively choose which tool applies to a situation, neither can an AI agent."

### Tool Design Principles

**1. Minimize Overlap**
```
❌ Bad: Three tools that can all "search files"
   - search_files()
   - find_in_project()
   - grep_codebase()

✅ Good: Clear separation
   - glob_files(pattern)      → Find by name/path pattern
   - grep_content(regex)      → Search file contents
   - semantic_search(query)   → AI-powered concept search
```

**2. Token-Efficient Returns**

Tools should return concise, structured information:

```python
# ❌ Returns full file content (bloats context)
def read_file(path):
    return open(path).read()

# ✅ Returns summary + option to expand
def read_file(path, lines=None):
    content = open(path).read()
    if lines:
        return extract_lines(content, lines)
    return {
        "path": path,
        "lines": count_lines(content),
        "preview": content[:500],
        "structure": extract_functions(content)
    }
```

**3. Unambiguous Parameters**

```python
# ❌ Ambiguous
def search(query, mode):  # What modes exist?

# ✅ Clear
def search_code(
    pattern: str,           # Regex pattern to match
    file_types: list[str],  # e.g., ["*.py", "*.js"]
    case_sensitive: bool = False
):
```

### Tool Set Size

Studies suggest optimal tool sets contain **5-15 tools**. More tools create:
- Decision paralysis for the model
- Increased context from tool descriptions
- Higher chance of tool confusion

---

## 5. Dynamic Context Retrieval

### Just-in-Time vs Pre-Loading

| Approach | Pros | Cons |
|----------|------|------|
| **Pre-loading** | Faster execution, no retrieval latency | Context pollution, stale data |
| **Just-in-time** | Fresh data, minimal context | Slower, requires good retrieval |

### The Progressive Disclosure Pattern

Instead of loading all relevant data upfront, agents can:

1. **Maintain lightweight identifiers** (file paths, query strings, API endpoints)
2. **Dynamically retrieve** data at runtime using tools
3. **Progressively discover** context through exploration

**Example: Claude Code's Approach**

Rather than pre-indexing an entire codebase, Claude Code:
```
1. Receives task: "Fix the authentication bug"
2. Searches for "auth" patterns → finds relevant files
3. Reads specific functions, not entire files
4. Discovers related code through imports
5. Builds understanding incrementally
```

This mirrors how humans investigate codebases—we don't memorize everything upfront.

### The Memory-Bank Pattern

**Source**: Production implementation from FanDuel nexus (Spark pipelines) and data-api (FastAPI platform)

The **memory-bank pattern** is the most important pattern for context engineering. Instead of embedding documentation in rules (bloating context), maintain a separate `memory-bank/` directory with indexed navigation.

**Structure**:
```
project/
├── .cursor/rules/         # Small: constraints + navigation pointers
├── .windsurf/rules/       # Small: constraints + navigation pointers  
├── CLAUDE.md              # Small: constraints + navigation pointers
└── memory-bank/
    ├── INDEX.md           # Navigation hub (< 100 lines)
    ├── components/        # Component docs
    ├── workflows/         # How-to guides
    └── architecture/      # Design docs
```

### 4-Level Documentation Hierarchy

The nexus memory-bank demonstrates progressive disclosure through documentation layers:

**Level 0: Index Files** (< 60 lines)
- Purpose: Navigation only
- Example: `memory-bank/INDEX.md`, `memory-bank/fd-nexus/INDEX.md`

**Level 1: Quick References** (< 100 lines)
- Purpose: Copy-paste solutions for common tasks
- Example: `quick-refs/common_transformations.md` - Top 10 transforms with examples

**Level 2: Component Overviews** (150-500 lines)
- Purpose: Understanding framework architecture
- Example: `components/transformations_overview.md`

**Level 3: Deep References** (500-1000+ lines)
- Purpose: Comprehensive component documentation
- Example: `features/state-pattern/state_pattern_implementation.md` (1,400 lines)

### File Size Annotations for Context Budgeting

Include explicit file size annotations to help choose docs based on context budget:

```markdown
## File Size Reference

**Lightweight (< 200 lines):** Quick reads
- `testing/tests_summary.md`, `components/kafka_api_transformation.md`

**Medium (200-500 lines):** Focused deep-dives
- `project_overview.md`, `components/readers_writers_overview.md`

**Large (500-1000 lines):** Comprehensive references
- `components/state_management_overview.md`, `testing/testing_overview.md`

**Very Large (> 1000 lines):** Detailed specifications
- `features/external-api-transformation/adr.md`, `features/state-pattern/...`
```

**Key insight**: This mirrors how humans learn—start with navigation, then quick solutions, then understanding, then deep references. The AI can choose the appropriate depth based on the task complexity.

### Task-Based INDEX Navigation

From data-api production implementation:

```markdown
## Navigation by Task

| Task | Start With | Size |
|------|------------|------|
| **Creating new API endpoint** | `workflows/adding-new-endpoint.md` | 365 lines |
| **Testing endpoints** | `workflows/testing-endpoints.md` | 360 lines |
| **Debugging/troubleshooting** | `workflows/troubleshooting.md` | 468 lines |
| **Code review** | `workflows/code-review-checklist.md` | 395 lines |
```

**Why this works:**
- Routes to the right doc based on **intent**, not structure
- Small enough to read in 2 minutes (< 200 lines for INDEX)
- Each task has a single "start here" file, avoiding analysis paralysis
- Line counts help budget context appropriately

---

## 6. Long-Horizon Task Techniques

For tasks spanning many turns or requiring extensive exploration, standard context management breaks down. Here are proven techniques:

### Compaction

**What it is**: Summarizing conversations approaching context limits while preserving critical information.

**Implementation Strategy**:
```
1. Maximize Recall First
   - Capture ALL potentially relevant information
   - Include architectural decisions
   - Note implementation details

2. Iterate to Improve Precision
   - Remove redundant code outputs
   - Summarize repetitive exchanges
   - Keep the 5 most recently accessed files in full

3. Preserve Anchors
   - File paths that were modified
   - Key decisions and their rationale
   - Error messages and their resolutions
```

**Claude Code's Approach**:
When conversations approach ~80% of context limit, Claude Code:
- Compresses message history
- Retains recent file contents
- Keeps all tool call results
- Summarizes earlier discussion

### Structured Note-Taking (Agentic Memory)

**What it is**: Agents write notes persisted OUTSIDE the context window, retrieving them when needed.

**Real-World Example: Claude Playing Pokémon**

Anthropic demonstrated Claude playing Pokémon across thousands of turns. The agent maintained:
- Objective tracking (current goal, subgoals)
- Strategic decisions (party composition, item usage)
- Map knowledge (previously visited locations)

This was impossible within a single context window—notes enabled continuity.

**Implementation**:
```markdown
# Memory System Rules

## Writing Notes
When you complete a significant step, write a note:
- Use write_note(category, content)
- Categories: "decisions", "context", "todo", "learnings"
- Be concise: 2-3 sentences max

## Reading Notes
Before starting a complex task:
- Use read_notes(category) to retrieve relevant context
- Prioritize "decisions" for architectural choices
- Check "todo" for pending items
```

### Sub-Agent Architecture

**What it is**: Specialized sub-agents handle focused tasks with clean context windows, returning condensed summaries.

**Pattern**:
```
Lead Agent (orchestrator)
├── Explorer Agent → Returns: "Found 3 relevant files, main logic in /auth/jwt.ts"
├── Analyzer Agent → Returns: "Bug caused by expired token check at line 45"
└── Implementer Agent → Returns: "Fixed in commit abc123, added 2 tests"
```

**Benefits**:
- Each agent gets a fresh, focused context
- Summaries (1,000-2,000 tokens) replace raw exploration
- Lead agent synthesizes without context pollution

**Trade-offs**:
- Higher latency (multiple LLM calls)
- Increased cost
- Requires good task decomposition

---

## 7. Practical Implementation

### Start Minimal

```markdown
# Anti-pattern: Kitchen sink approach
[50+ rules, every edge case, exhaustive examples]

# Better: Iterative refinement
1. Start with 5-10 core rules
2. Test on real tasks
3. Add rules ONLY when you observe failures
4. Remove rules that aren't triggered
```

### Organize Ruthlessly

**Hierarchy of Context Sources**:
```
1. System Prompt (always present)
   └── Core identity, constraints, response format

2. Project Rules (loaded per-project)
   └── Tech stack, architecture, conventions

3. File-Specific Rules (loaded with files)
   └── Component patterns, test conventions

4. Dynamic Context (retrieved on-demand)
   └── Documentation, error messages, search results
```

### Testing and Iteration

**Create a Context Test Suite**:
```markdown
## Test Cases for Context Quality

1. **Ambiguity Test**
   - Give the agent an ambiguous task
   - Does it ask clarifying questions or guess?

2. **Recall Test**
   - Reference information from early in context
   - Can the agent retrieve it accurately?

3. **Tool Selection Test**
   - Present a task that could use multiple tools
   - Does it choose the right one consistently?

4. **Degradation Test**
   - Run a long session (50+ turns)
   - Does response quality degrade? When?
```

### Key Metrics

Track these to measure context effectiveness:

| Metric | What It Measures |
|--------|------------------|
| **First-attempt success rate** | Is context sufficient for task completion? |
| **Clarification frequency** | Is context clear or ambiguous? |
| **Context utilization** | How much of provided context is referenced? |
| **Token efficiency** | Results achieved per token spent |

**Target**: Keep context utilization under 40% during implementation phases.

---

## Summary: Context Engineering Principles

1. **Context is finite** — Treat it as a precious resource with diminishing marginal returns

2. **Signal over volume** — A smaller, high-quality context outperforms a larger, diluted one

3. **Dynamic over static** — Retrieve information when needed rather than pre-loading everything

4. **Structure enables retrieval** — Organized context (tags, headers, delimiters) is more accessible

5. **Iterate based on failures** — Add complexity only when simple approaches fail

6. **Let models be intelligent** — Capable models can explore and reason; don't over-specify

---

## References

- [Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) - Anthropic
- [The Rise of Context Engineering](https://blog.langchain.com/the-rise-of-context-engineering/) - LangChain
- [Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices) - Anthropic

**Production Implementations**:
- FanDuel nexus (Spark pipelines) - Memory-bank pattern with 4-level hierarchy
- Data API Platform (FastAPI) - Task-based navigation with file size annotations
