# AI Development Tools: Comparison Guide

> **Key insight**: Context engineering principles are tool-agnostic. The same strategy works across all tools—only the format differs.

**Attribution**: Synthesized from production implementations in nexus, data-api, and comparative analysis.

---

## Side-by-Side Feature Comparison

| Feature | Cursor | Windsurf | Claude Code | GitHub Copilot |
|---------|--------|----------|-------------|----------------|
| **Config Format** | `.cursor/rules/*.mdc` | `.windsurf/rules/*.md` | `CLAUDE.md` | `.github/copilot-instructions.md` |
| **Frontmatter** | YAML required | None | None | None |
| **Multiple Files** | ✅ Multiple rules | ✅ Multiple rules | ✅ Hierarchy | ❌ Single file |
| **Conditional Loading** | ✅ `globs: ["pattern"]` | ❌ All rules load | ✅ Directory hierarchy | ❌ Always loaded |
| **Always-Apply Rules** | ✅ `alwaysApply: true` | ✅ All rules (implicit) | ✅ Project CLAUDE.md | ✅ Always |
| **Rule Size Target** | < 100 lines | < 75 lines | < 150 lines | < 200 lines |
| **Context Commands** | New chat | New conversation | `/clear` | New chat |
| **Extended Thinking** | No | No | ✅ `think`, `think harder` | No |
| **Cost Monitoring** | Via UI | Via UI | `/cost` command | Via UI |
| **Custom Commands** | No | No | ✅ `.claude/commands/` | No |
| **File Browser** | ✅ Integrated | ✅ Integrated | Terminal-based | ✅ Integrated |
| **IDE Support** | Cursor IDE | Windsurf IDE | Terminal (any IDE) | All (VS Code, JetBrains, etc.) |
| **Best For** | Large projects, precise control | Rapid prototyping | Terminal workflows, long tasks | Team consistency, GitHub integration |
| **Skills** | No | No | ✅ `.claude/skills/` | No |
| **Sub-Agents** | No | No | ✅ `.claude/agents/` | No |
| **Hooks** | No | No | ✅ 13 events | No |
| **Modular Rules** | ✅ `.cursor/rules/` | ✅ `.windsurf/rules/` | ✅ `.claude/rules/` | No |
| **Plugins** | ✅ Extensions | ✅ Extensions | ✅ Plugins | ✅ Extensions |
| **MCP Support** | ✅ | ✅ | ✅ `.claude/settings.json` | Limited |
| **Memory-Bank Support** | ✅ Excellent | ✅ Excellent | ✅ Excellent | ✅ Excellent |
| **Pricing Model** | IDE license + API | IDE license + API | API usage only | GitHub subscription |

---

## Configuration Format Differences

### Cursor: .mdc Files with YAML

**Location**: `.cursor/rules/`

**Format**:
```yaml
---
description: Project rules
globs: ["project/**/*"]
---

# Markdown Content
```

**Pros**:
- Conditional loading via globs
- Fine-grained control
- Metadata in frontmatter

**Cons**:
- More complex format
- YAML syntax required

### Windsurf: Plain Markdown

**Location**: `.windsurf/rules/`

**Format**:
```markdown
# Markdown Content

No frontmatter needed.
```

**Pros**:
- Simple format
- No YAML to manage
- Quick to set up

**Cons**:
- No conditional loading
- All rules always active
- Must be extra concise

### Claude Code: Single File + Hierarchy

**Location**: Project root, subdirectories, or global

**Format**:
```markdown
# CLAUDE.md

Plain Markdown, no frontmatter.
```

**Pros**:
- Directory-based scoping
- Extended thinking modes
- Workflow-focused
- Custom commands

**Cons**:
- Less granular than Cursor globs
- Single file per directory
- Requires terminal comfort

### GitHub Copilot: Single Repository File

**Location**: `.github/copilot-instructions.md`

**Format**:
```markdown
# GitHub Copilot Custom Instructions

Plain Markdown, no frontmatter.
```

**Pros**:
- Native GitHub integration
- Works across all IDEs
- Simple single-file setup
- Team-wide consistency

**Cons**:
- No conditional loading
- Single file only
- Always loaded (must be concise)
- Repository-scoped only

---

## Context Loading Mechanisms

### Cursor: Pattern-Based Loading

```yaml
# workspace.mdc - Always loaded
---
alwaysApply: true
---

# frontend.mdc - Loaded when editing frontend/**/*
---
globs: ["frontend/**/*"]
---

# backend.mdc - Loaded when editing backend/**/*
---
globs: ["backend/**/*"]
---
```

**Result**: Precise control, minimal context footprint.

**Use case**: Large monorepos with distinct projects.

### Windsurf: All-or-Nothing Loading

```
.windsurf/rules/
├── workspace.md   ← Always loaded
├── frontend.md    ← Always loaded
└── backend.md     ← Always loaded
```

**Result**: All rules loaded for every session.

**Use case**: Smaller projects, or very concise rules.

### Claude Code: Hierarchical Loading

```
~/.claude/CLAUDE.md              ← Global (always)
/project/CLAUDE.md               ← Project (when in project)
/project/backend/CLAUDE.md       ← Directory (when in backend/)
```

**Result**: Directory-scoped, hierarchical override.

**Use case**: Terminal workflows, phased development.

### GitHub Copilot: Single File Loading

```
.github/
└── copilot-instructions.md      ← Always loaded (repository-wide)
```

**Result**: Single file always loaded for entire repository.

**Use case**: Team-wide consistency, cross-IDE support.

---

## When to Use Which Tool

### Use Cursor When...

✅ **Large, complex projects** with multiple sub-projects
✅ **Need precise control** over what loads when
✅ **Team prefers IDE-based development**
✅ **Want file-pattern based rule activation**

**Example**: Monorepo with 5 distinct services, each with different tech stacks.

### Use Windsurf When...

✅ **Rapid prototyping** or smaller projects
✅ **Simplicity over granular control**
✅ **Team wants minimal config overhead**
✅ **Rules are naturally small** (< 75 lines each)

**Example**: Single-service API, straightforward stack.

### Use Claude Code When...

✅ **Terminal-based workflows** preferred
✅ **Long-horizon tasks** (research → plan → implement)
✅ **Extended thinking** needed for complex problems
✅ **Want custom slash commands**
✅ **Active context management** is priority

**Example**: Complex migrations, architectural refactoring, exploration tasks.

### Use GitHub Copilot When...

✅ **Team uses multiple IDEs** (VS Code, JetBrains, Neovim, etc.)
✅ **Native GitHub integration** desired
✅ **Simple, single-file setup** preferred
✅ **Repository-wide consistency** is priority
✅ **Complementing other tools** (works alongside Cursor/Windsurf/Claude)

**Example**: Open-source projects, teams with diverse tooling, GitHub-centric workflows.

---

## Can You Use Multiple Tools Together?

**Yes!** And it's recommended for team flexibility.

### Shared Foundation: Memory-Bank

```
project/
├── .github/
│   └── copilot-instructions.md  ← GitHub Copilot format
├── .cursor/rules/               ← Cursor-specific format
├── .windsurf/rules/             ← Windsurf-specific format
├── CLAUDE.md                    ← Claude Code format
└── memory-bank/                 ← SHARED by all tools
    └── INDEX.md
```

**Key insight**: Memory-bank is tool-agnostic, so maintain one set of docs.

### Maintenance Strategy

1. **Pick a primary tool** (e.g., Cursor or GitHub Copilot)
2. **Write rules for primary tool**
3. **Adapt to other formats** (95% same content)
4. **Share memory-bank** across all tools

**Example Workflow**:

```bash
# 1. Write Cursor rules
vim .cursor/rules/workspace.mdc

# 2. Convert to Windsurf (strip YAML)
sed '/^---$/,/^---$/d' .cursor/rules/workspace.mdc > .windsurf/rules/workspace.md

# 3. Adapt to Claude Code (consolidate to CLAUDE.md)
# Manual: combine key sections

# 4. Memory-bank stays identical
# No changes needed - already tool-agnostic
```

### Real-World Example

**data-api project** supports all three:

```
data-api/
├── .cursor/rules/
│   └── workspace.mdc (44 lines)
├── .windsurf/rules/
│   └── workspace.md (39 lines, same content)
├── CLAUDE.md (266 lines, includes commands)
└── memory-bank/ (21 files, ~6,049 lines)
    └── INDEX.md
```

**Content overlap**:
- Cursor workspace: 44 lines
- Windsurf workspace: 39 lines (95% same)
- Claude CLAUDE.md: 266 lines (includes dev commands)
- Memory-bank: **100% shared**

---

## Migration Paths Between Tools

### Cursor → Windsurf

**Easy** (same content, different format):

```bash
# For each .mdc file
for file in .cursor/rules/*.mdc; do
    name=$(basename "$file" .mdc)
    
    # Strip YAML frontmatter
    sed '/^---$/,/^---$/d' "$file" > ".windsurf/rules/${name}.md"
    
    # Add scope note
    echo "**Scope**: [add manually based on globs]" >> ".windsurf/rules/${name}.md"
done
```

### Cursor → Claude Code

**Moderate** (consolidate rules into CLAUDE.md):

1. Copy common sections from workspace.mdc
2. Add development commands
3. Add memory-bank navigation
4. Reference project-specific rules in memory-bank

**Result**: Single CLAUDE.md (< 150 lines) + memory-bank.

### Windsurf → Cursor

**Easy** (add YAML frontmatter):

```bash
# For each .md file
for file in .windsurf/rules/*.md; do
    name=$(basename "$file" .md)
    
    # Add frontmatter
    cat > ".cursor/rules/${name}.mdc" <<EOF
---
description: [Add description]
alwaysApply: true  # or globs: [...]
---

$(cat "$file")
EOF
done
```

### Claude Code → Cursor/Windsurf

**Moderate** (split CLAUDE.md into focused rules):

1. Extract workspace overview → workspace rule
2. Extract project-specific sections → project rules
3. Move detailed docs → memory-bank
4. Add appropriate frontmatter (Cursor) or format as plain MD (Windsurf)

---

## Cost Considerations

### Cursor

**Model**: IDE subscription + API usage

**Typical costs**:
- IDE: ~$20/month (includes AI features)
- API: Pay-as-you-go for Claude/GPT calls

**Best for**: Teams with IDE budget, moderate AI usage.

### Windsurf

**Model**: IDE subscription + API usage

**Typical costs**:
- IDE: ~$15-25/month (includes AI features)
- API: Pay-as-you-go for Claude/GPT calls

**Best for**: Similar to Cursor, preference based.

### Claude Code

**Model**: API usage only (no IDE license)

**Typical costs**:
- No IDE fee
- API: Anthropic Claude API rates
  - Sonnet: ~$3/million input tokens
  - Opus: ~$15/million input tokens

**Best for**:
- Terminal users (avoid IDE costs)
- Teams controlling AI spend directly
- Long-thinking tasks (Opus)

### Cost Optimization for All

**1. Small rules** (< 100 lines)
- Less context loaded per session
- Fewer tokens spent

**2. Memory-bank pattern**
- Retrieve only needed docs
- Avoid pre-loading everything

**3. Context clearing**
- Fresh context between tasks
- Avoid accumulated bloat

**4. Extended thinking (Claude Code)**
- Invest thinking upfront → fewer iterations
- Better plans → less wasted implementation

**Cost Example** (Claude Sonnet):
```
❌ Bad approach:
   - Load all docs upfront: 50,000 tokens
   - Iterate 5 times: 250,000 tokens
   - Cost: ~$0.75

✅ Good approach:
   - Load minimal rules: 5,000 tokens
   - Retrieve 2 relevant docs: 3,000 tokens
   - Ultrathink upfront: 2,000 tokens
   - Implement correctly first time: 10,000 tokens
   - Cost: ~$0.06 (12x cheaper!)
```

---

## Recommendation Matrix

| Project Type | Primary Tool | Secondary Tool | Reason |
|--------------|--------------|----------------|--------|
| **Large monorepo** | Cursor | Claude Code | Cursor's globs + Claude's thinking for complex tasks |
| **Small service** | Windsurf or Copilot | - | Simplicity wins |
| **Terminal-heavy workflow** | Claude Code | Cursor | Terminal primary, IDE for quick edits |
| **Exploration/research** | Claude Code | - | Extended thinking shines |
| **Team with mixed IDEs** | GitHub Copilot | Cursor/Windsurf | Works across VS Code, JetBrains, etc. |
| **Team with mixed preferences** | All four | - | Memory-bank enables all |
| **Tight budget** | Claude Code or Copilot | - | No IDE license cost |
| **GitHub-centric teams** | GitHub Copilot | - | Native integration, PR context |

---

## Practical Comparison: Same Task, Three Tools

**Task**: Add a new API endpoint following existing patterns

### Cursor Approach

1. **Rules auto-load** when editing `src/api/` files
2. **Check memory-bank**: `memory-bank/INDEX.md` → `workflows/adding-endpoint.md`
3. **Read pattern**: Existing endpoint file
4. **Implement**: Follow pattern, rules provide constraints

**Strengths**:
- Auto-context loading (no manual `/add`)
- Precise rule activation

**Time**: ~10 minutes

### Windsurf Approach

1. **Rules always loaded** (all workspace + API rules)
2. **Check memory-bank**: `memory-bank/INDEX.md` → `workflows/adding-endpoint.md`
3. **Read pattern**: Existing endpoint file
4. **Implement**: Follow pattern, rules provide constraints

**Strengths**:
- Simpler setup
- Identical workflow to Cursor

**Time**: ~10 minutes

### Claude Code Approach

1. **Start fresh**: `claude /clear`
2. **Research phase**: "Research API endpoint patterns, create research file"
3. **Plan phase**: "Create implementation plan for new endpoint"
4. **Review plan**: Human verification
5. **Implement**: "Execute plan, keep context under 40%"

**Strengths**:
- Structured workflow
- Plan review before implementation
- Extended thinking for complex decisions

**Time**: ~15 minutes (more upfront planning, less iteration)

**When Claude Code wins**:
- Endpoint is complex or novel
- Team values spec-first approach
- Want to document architectural decisions

**When Cursor/Windsurf win**:
- Endpoint is straightforward
- Pattern is well-established
- Speed is priority

---

## Summary

### Tool Strengths

**Cursor**:
- ✅ Precise control (glob patterns)
- ✅ Large projects
- ✅ IDE integration
- ❌ More complex config

**Windsurf**:
- ✅ Simple setup
- ✅ Quick to start
- ✅ IDE integration
- ❌ Less granular control

**Claude Code**:
- ✅ Workflow-focused
- ✅ Extended thinking
- ✅ Terminal native
- ✅ Custom commands
- ❌ Less IDE integration

**GitHub Copilot**:
- ✅ Cross-IDE support
- ✅ GitHub integration
- ✅ Simple setup
- ✅ Team consistency
- ❌ No conditional loading
- ❌ Single file limitation

### Universal Truth

**All four work excellently with the memory-bank pattern.**

The choice comes down to:
1. Team workflow preferences (IDE vs terminal)
2. Project complexity (need for granular control)
3. Budget considerations (IDE licenses)
4. IDE diversity (single IDE vs multiple)

**Recommendation**: Start with one, keep memory-bank tool-agnostic, add others as needed.

---

## References

- See individual tool guides: `docs/tools/cursor-guide.md`, `windsurf-guide.md`, `claude-code-guide.md`, `copilot-guide.md`
- See `docs/fundamentals/memory-bank-pattern.md` for the shared foundation
- See `examples/` for real-world implementations across all tools
