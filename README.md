# Effective Context Engineering

> **A comprehensive reference repository for context engineering across AI development tools (Cursor, Windsurf, Claude Code), based on proven patterns from production projects.**

Transform how you work with AI assistants through battle-tested context engineering patterns that maximize AI effectiveness while minimizing context bloat.

---

## 🎯 Why Context Engineering?

**The Problem**: AI coding assistants are powerful, but without proper context configuration they:
- Make wrong assumptions instead of asking questions
- Miss existing documentation and patterns
- Hit context limits quickly
- Degrade in quality during long sessions
- Require constant repetition of project context

**The Solution**: **Context engineering** - building dynamic systems to provide the right information at the right time, enabling AI to accomplish tasks effectively.

**The Result**: AI that:
- ✅ Follows your project patterns consistently
- ✅ References documentation before asking questions
- ✅ Stays under 40% context utilization during work
- ✅ Maintains quality across long sessions
- ✅ Gets tasks right on first attempt (> 70% success rate)

---

## 🚀 Quick Start

### 3-Step Setup (< 30 minutes)

#### 1. Choose Your Tool

| Tool | Best For | Setup Time |
|------|----------|------------|
| **Cursor** | Large projects, precise control | 20 min |
| **Windsurf** | Rapid prototyping, simplicity | 15 min |
| **Claude Code** | Terminal workflows, long tasks | 10 min |
| **All Three** | Team flexibility | 30 min |

#### 2. Create Memory-Bank

```bash
# Create structure
mkdir -p memory-bank/{architecture,components,workflows}

# Copy template
cp templates/memory-bank/INDEX.md.template memory-bank/INDEX.md

# Edit and add your first 2-3 docs
vim memory-bank/INDEX.md
```

#### 3. Create Rules/Config

**For Cursor**:
```bash
mkdir -p .cursor/rules
cp templates/cursor/.cursor/rules/workspace.mdc.template .cursor/rules/workspace.mdc
# Edit workspace.mdc, fill in placeholders
```

**For Windsurf**:
```bash
mkdir -p .windsurf/rules
cp templates/windsurf/.windsurf/rules/workspace.md.template .windsurf/rules/workspace.md
# Edit workspace.md, fill in placeholders
```

**For Claude Code**:
```bash
cp templates/claude/CLAUDE.md.template CLAUDE.md
# Edit CLAUDE.md, fill in placeholders
```

**Done!** Your AI assistant now has effective context.

---

## 📚 What's Inside

### Documentation

#### Fundamentals
- **[Context Engineering](docs/fundamentals/context-engineering.md)** - Core concepts, context limitations, the evolution from prompt to context engineering
- **[Prompt Engineering](docs/fundamentals/prompt-engineering.md)** - Effective task prompts, examples by type, anti-patterns
- **[Memory-Bank Pattern](docs/fundamentals/memory-bank-pattern.md)** - **The most important pattern** - external documentation for AI-optimized retrieval

#### Tool Guides
- **[Cursor Guide](docs/tools/cursor-guide.md)** - `.cursor/rules/*.mdc` configuration, glob patterns, best practices
- **[Windsurf Guide](docs/tools/windsurf-guide.md)** - `.windsurf/rules/*.md` configuration, differences from Cursor
- **[Claude Code Guide](docs/tools/claude-code-guide.md)** - `CLAUDE.md` hierarchy, skills, agents, hooks, commands, workflows
- **[Claude Code Advanced](docs/tools/claude-code-advanced.md)** - Command-Skill-Agent architecture, RPI workflow, MCP, hooks
- **[Claude Code Settings Reference](docs/tools/claude-code-settings-reference.md)** - Complete settings.json reference (permissions, hooks, MCP, 40+ env vars)
- **[Claude Code Monorepo Guide](docs/tools/claude-code-monorepo.md)** - CLAUDE.md loading semantics, skills discovery, hierarchy patterns
- **[Tool Comparison](docs/tools/tool-comparison.md)** - Side-by-side feature comparison, when to use which

#### Best Practices
- **[Rule Writing](docs/best-practices/rule-writing.md)** - Start minimal, iterate based on failures, keep rules under 100 lines
- **[Anti-Patterns](docs/best-practices/anti-patterns.md)** - Common mistakes to avoid (kitchen sink, generic advice, pre-loading, LLMs as linters)
- **[Context Budgeting](docs/best-practices/context-budgeting.md)** - File size annotations, progressive disclosure, target 40-60% with FIC
- **[Practical Tips](docs/best-practices/practical-tips.md)** - Real-world tips from production: workflow, tooling, debugging, prompts

#### Workflows
- **[Spec-First Development](docs/workflow/spec-first-development.md)** - Research → Plan → Implement methodology
- **[Long-Horizon Tasks](docs/workflow/long-horizon-tasks.md)** - Compaction, structured notes, sub-agents
- **[Testing Context](docs/workflow/testing-context.md)** - How to test and measure context effectiveness

### Templates

Ready-to-use templates for all three tools:

- **[Cursor Templates](templates/cursor/)** - `workspace.mdc`, `project.mdc` with YAML frontmatter
- **[Windsurf Templates](templates/windsurf/)** - `workspace.md`, `project.md` plain Markdown
- **[Claude Code Templates](templates/claude/)** - `CLAUDE.md`, `settings.json`, skills, agents, commands, hooks, rules, RPI workflow
- **[Memory-Bank Template](templates/memory-bank/)** - `INDEX.md` with task-based navigation

### Examples

Real-world examples based on production projects:

- **[Simple Project](examples/simple-project/)** - Minimal setup for single-project repos
- **[FastAPI Microservice](examples/fastapi-microservice/)** - Based on data-api platform
- **[Monorepo Data Platform](examples/monorepo-data-platform/)** - Based on FanDuel nexus

### Checklists

Quick reference guides:

- **[Setup Checklist](checklists/setup-checklist.md)** - Step-by-step setup guide
- **[Rule Review Checklist](checklists/rule-review-checklist.md)** - Quality check for rules
- **[Context Health Check](checklists/context-health-check.md)** - Measure effectiveness

### References

- **[Resources](references/RESOURCES.md)** - Curated links to articles, videos, documentation
- **[Research Notes](references/research-notes.md)** - Learnings from production implementations

---

## 🔑 Key Concepts

### The Memory-Bank Pattern

**The foundation of effective context engineering**:

```
project/
├── .cursor/rules/         # Small constraints (< 100 lines each)
├── .windsurf/rules/       # Small constraints (< 75 lines each)
├── CLAUDE.md              # Small constraints (< 150 lines)
└── memory-bank/           # Detailed documentation (any size)
    ├── INDEX.md           # Navigation hub (< 100 lines)
    ├── architecture/
    ├── components/
    └── workflows/
```

**Why it works**:
- Rules stay minimal → low context footprint
- AI retrieves docs just-in-time → high signal-to-noise
- Single source of truth → no duplication
- Tool-agnostic → same memory-bank for all tools

### Task-Based Navigation

Don't organize by structure, organize by intent:

```markdown
## Navigation by Task

| Task | Start With | Size |
|------|------------|------|
| Creating new endpoint | `workflows/adding-endpoint.md` | 365 lines |
| Debugging issues | `workflows/troubleshooting.md` | 468 lines |
| Understanding architecture | `architecture/overview.md` | 87 lines |
```

AI knows **what you want to do**, routes to the right doc.

### File Size Annotations

Help AI budget context appropriately:

```markdown
## File Size Reference

### Lightweight (< 150 lines) - Quick reads
- Token cost: ~500 tokens
- Use for: Quick lookups, syntax

### Medium (150-350 lines) - Focused deep-dives
- Token cost: ~1,000-1,500 tokens
- Use for: Understanding components

### Large (350-500 lines) - Comprehensive references
- Token cost: ~1,500-2,000 tokens
- Use for: Complex workflows, troubleshooting
```

AI chooses appropriate depth based on task and context budget.

---

## 📊 Real-World Impact

### Before Context Engineering

```
First-attempt success: 45%
Context utilization: 65% (before starting work)
AI asks questions: 15% (makes assumptions instead)
Time to onboard: 2 weeks
```

### After Context Engineering

```
First-attempt success: 78%
Context utilization: 12% (before starting work)
AI asks questions: 35% (appropriately cautious)
Time to onboard: 1 day
```

**Source**: Metrics from nexus and data-api production implementations

---

## 🛠️ Tool-Agnostic Approach

**Core insight**: Context engineering principles are universal. Only the format differs.

| Aspect | Cursor | Windsurf | Claude Code |
|--------|--------|----------|-------------|
| **Rules location** | `.cursor/rules/*.mdc` | `.windsurf/rules/*.md` | `CLAUDE.md` |
| **Format** | YAML + Markdown | Plain Markdown | Plain Markdown |
| **Content** | 95% identical | 95% identical | 95% identical |
| **Memory-bank** | Shared | Shared | Shared |

**Recommendation**: Support all three tools for team flexibility. Memory-bank is 100% shared, rules are 95% shared.

---

## 📖 Directory Structure

```
effective-context-engineering/
├── README.md                          # This file
├── docs/                              # Core documentation
│   ├── fundamentals/                  # Context engineering concepts
│   ├── tools/                         # Tool-specific guides
│   │   ├── claude-code-guide.md       # Claude Code basics + skills, agents, hooks
│   │   ├── claude-code-advanced.md    # Advanced patterns (RPI, MCP, hooks)
│   │   ├── claude-code-settings-reference.md  # Complete settings reference
│   │   ├── claude-code-monorepo.md    # Monorepo-specific guidance
│   │   ├── cursor-guide.md            # Cursor configuration
│   │   ├── windsurf-guide.md          # Windsurf configuration
│   │   └── tool-comparison.md         # Side-by-side comparison
│   ├── best-practices/                # Rule writing, anti-patterns
│   │   └── practical-tips.md          # Real-world production tips
│   └── workflow/                      # Development workflows
├── templates/                         # Ready-to-use templates
│   ├── cursor/                        # Cursor templates
│   ├── windsurf/                      # Windsurf templates
│   ├── claude/                        # Claude Code templates
│   │   ├── CLAUDE.md.template         # WHY/WHAT/HOW framework
│   │   ├── settings.json.template     # Team settings
│   │   ├── settings.local.json.template # Personal settings
│   │   ├── skills/                    # Skill templates
│   │   ├── agents/                    # Agent templates
│   │   ├── commands/                  # Command templates
│   │   ├── hooks/                     # Hooks setup + examples
│   │   ├── rules/                     # Modular rule templates
│   │   └── rpi/                       # Research-Plan-Implement workflow
│   └── memory-bank/                   # Memory-bank template
├── examples/                          # Real-world examples
│   ├── simple-project/                # Minimal example (with .claude/)
│   ├── fastapi-microservice/          # API platform example
│   └── monorepo-data-platform/        # Monorepo example
├── checklists/                        # Quick reference guides
│   ├── setup-checklist.md
│   ├── rule-review-checklist.md
│   └── context-health-check.md
└── references/                        # External resources
    ├── RESOURCES.md                   # Curated links (updated Feb 2026)
    └── research-notes.md              # Production learnings (20 tips)
```

---

## 🎓 Learning Path

### Beginner (1-2 hours)

1. Read [Memory-Bank Pattern](docs/fundamentals/memory-bank-pattern.md)
2. Read [Rule Writing Best Practices](docs/best-practices/rule-writing.md)
3. Follow [Setup Checklist](checklists/setup-checklist.md)
4. Use templates to create your first config

### Intermediate (2-4 hours)

1. Read [Context Engineering](docs/fundamentals/context-engineering.md)
2. Study [Real-World Examples](examples/)
3. Implement [Spec-First Development](docs/workflow/spec-first-development.md)
4. Review [Anti-Patterns](docs/best-practices/anti-patterns.md)

### Advanced (4-8 hours)

1. Read [Long-Horizon Tasks](docs/workflow/long-horizon-tasks.md)
2. Implement [Context Testing](docs/workflow/testing-context.md)
3. Study [Context Budgeting](docs/best-practices/context-budgeting.md)
4. Read [Claude Code Advanced](docs/tools/claude-code-advanced.md) — skills, agents, hooks, MCP
5. Set up [RPI Workflow](templates/claude/rpi/) — Research-Plan-Implement commands
6. Review [Practical Tips](docs/best-practices/practical-tips.md) and [Research Notes](references/research-notes.md)

---

## 🤝 Contributing

Contributions welcome! This repository benefits from:

- **Production examples** from your projects
- **Lessons learned** from implementations
- **Tool-specific tips** and tricks
- **Documentation improvements**

### How to Contribute

1. Fork the repository
2. Create a feature branch
3. Add your contribution
4. Submit a pull request

### Areas Needing Contributions

- [ ] Examples from different tech stacks (Go, Rust, Java, etc.)
- [ ] Industry-specific patterns (ML, DevOps, Mobile)
- [ ] Additional testing patterns
- [ ] Tool comparison updates for new AI tools

---

## 📜 License

This repository is provided as-is for educational and reference purposes.

Individual code snippets and examples may be used freely. Please attribute when sharing substantial portions.

---

## 🙏 Attribution

This repository synthesizes knowledge and patterns from production projects:

- **FanDuel nexus** - Enterprise Spark library for Databricks demonstrating memory-bank pattern at scale (50+ files, 4-level hierarchy)
- **Data API Platform** - FastAPI self-service platform with task-based navigation and file size annotations (21 files, task-optimized)
- **ai-presentation** - Context engineering research compilation and prompt engineering patterns

Special thanks to the teams behind these projects for pioneering effective context engineering patterns.

---

## 📞 Support and Community

- **Issues**: Report bugs or request features via [GitHub Issues](https://github.com/yourusername/effective-context-engineering/issues)
- **Discussions**: Share experiences and ask questions via [GitHub Discussions](https://github.com/yourusername/effective-context-engineering/discussions)

---

## 🎯 Success Criteria

You'll know your context engineering is effective when:

- ✅ First-attempt success rate > 70% for well-defined tasks
- ✅ AI references memory-bank before asking questions
- ✅ AI consistently follows documented patterns
- ✅ AI avoids documented anti-patterns
- ✅ Context utilization stays under 40% during work
- ✅ Quality remains high across 50+ turn sessions
- ✅ New team members productive in < 1 day
- ✅ Rules stay under 100 lines each

---

## 🚀 Next Steps

1. **Start with Quick Start** (above) - 30 minutes to working config
2. **Read Memory-Bank Pattern** - Understand the foundation
3. **Use Templates** - Don't start from scratch
4. **Test with Real Tasks** - Verify effectiveness
5. **Iterate Based on Failures** - Add rules only when you observe problems
6. **Share Learnings** - Contribute back to this repository

---

**Ready to transform your AI-assisted development? Start with the [Quick Start](#-quick-start) above!**
