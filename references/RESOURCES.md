# AI Development Resources

Curated resources for context engineering and AI-assisted development across tools.

---

## Context Engineering (Essential Reading)

### Core Concepts

- **[Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)** - Anthropic  
  The definitive guide covering context rot, compaction, sub-agents, and practical techniques.

- **[The Rise of Context Engineering](https://blog.langchain.com/the-rise-of-context-engineering/)** - LangChain  
  Why context engineering supersedes prompt engineering and how to think about dynamic context.

- **[Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)** - Anthropic  
  Official best practices including CLAUDE.md usage, workflows, and effective context management.

### Video Content

- **[Context Engineering: The New Frontier in AI](https://www.youtube.com/watch?v=nkOLuILaOTo&t=70s)**  
  Video overview of context engineering concepts and practical applications.

- **[Advanced Context Engineering for AI Workflows](https://www.youtube.com/watch?v=IS_y40zY-hc)**  
  Deep dive into advanced techniques for long-horizon tasks and context management.

---

## Tool-Specific Documentation

### Cursor

- **[Cursor Rules Documentation](https://cursor.com/docs/context/rules)** - Official  
  Complete reference for .mdc files, rule types, and configuration.

- **[Top Cursor Rules for Coding Agents](https://www.prompthub.us/blog/top-cursor-rules-for-coding-agents)**  
  Practical examples of effective rules from real projects.

- **[Cursor IDE Rules for AI](https://kirill-markin.com/articles/cursor-ide-rules-for-ai/)**  
  Community guidelines and patterns for effective Cursor configuration.

- **[Cursor Best Practices GitHub](https://github.com/digitalchild/cursor-best-practices)**  
  Community-maintained repository of best practices and examples.

- **[Cursor AI Complete Guide 2025](https://medium.com/@hilalkara.dev/cursor-ai-complete-guide-2025-real-experiences-pro-tips-mcps-rules-context-engineering-6de1a776a8af)**  
  Comprehensive guide with real experiences, tips, and context engineering patterns.

### Windsurf

- **[Windsurf Documentation](https://docs.windsurf.com/)** - Official  
  Complete documentation for Windsurf IDE and AI features.

- **[Windsurf AI Rules Guide](https://uibakery.io/blog/windsurf-ai-rules)**  
  Detailed guide on prompting and rules configuration for Windsurf.

- **[AI Coding Assistant Rules for Windsurf and Cursor](https://deeplearning.fr/ai-coding-assistant-rules-for-windsurf-and-cursor/)**  
  Cross-tool comparison and configuration patterns.

- **[Windsurf Rules That Actually Work](https://playbooks.com/windsurf-rules)**  
  Practical, tested patterns from production use.

### Claude Code

- **[Using CLAUDE.md Files](https://www.claude.com/blog/using-claude-md-files)** - Official
  Official guide on CLAUDE.md configuration and best practices.

- **[Claude Code Skills Documentation](https://docs.anthropic.com/en/docs/claude-code/skills)** - Official
  Skills system: AI-discoverable capabilities with YAML frontmatter.

- **[Claude Code Hooks Documentation](https://docs.anthropic.com/en/docs/claude-code/hooks)** - Official
  Hooks system: shell commands triggered by Claude Code events.

- **[Claude Code Sub-Agents Documentation](https://docs.anthropic.com/en/docs/claude-code/sub-agents)** - Official
  Autonomous sub-agents with Task tool invocation.

- **[Claude Code Settings Documentation](https://docs.anthropic.com/en/docs/claude-code/settings)** - Official
  4-tier settings hierarchy, permissions, MCP configuration.

- **[Claude Code MCP Documentation](https://docs.anthropic.com/en/docs/claude-code/mcp)** - Official
  Model Context Protocol server configuration and usage.

- **[CLAUDE.md Best Practices](https://arize.com/blog/claude-md-best-practices-learned-from-optimizing-claude-code-with-prompt-learning/)**
  Optimization patterns and lessons learned from production usage.

- **[How I Use Every Claude Code Feature](https://blog.sshh.io/p/how-i-use-every-claude-code-feature)**
  Comprehensive walkthrough of all Claude Code features and workflows.

- **[Claude Code Complete Guide](https://www.siddharthbharath.com/claude-code-the-complete-guide/)**
  End-to-end guide covering setup, workflows, and advanced techniques.

### Claude Code Workflows & Configuration (2026)

- **[HumanLayer CLAUDE.md Guide](https://humanlayer.dev/blog/claude-md)** — HumanLayer
  WHY/WHAT/HOW framework for structuring CLAUDE.md files.

- **[ACE-FCA Context Engineering Framework](https://github.com/davehague/ace-fca)** — Community
  Automated Context Engineering with Frequent Compaction Architecture.

- **[Boris Cherny Feb 2026 Workflow](https://borisCherny.com/claude-code-workflow)** — Community
  Production workflow patterns for Claude Code in 2026.

- **[Shipyard Claude Code Cheatsheet](https://shipyard.build/blog/claude-code-cheatsheet)** — Shipyard
  Quick reference for Claude Code commands, settings, and workflows.

- **[ClaudeLog Environment Variables](https://claudelog.com/env-vars)** — Community
  Comprehensive guide to 40+ Claude Code environment variables.

- **[Eesel AI Settings Guide](https://eesel.ai/blog/claude-code-settings)** — Eesel AI
  Detailed settings.json configuration guide with examples.

- **[GitHub Spec-Kit](https://github.com/spec-kit/spec-kit)** — Community
  Specification-first development kit for Claude Code projects.

- **[Andrej Karpathy Skills Examples](https://github.com/karpathy/claude-skills)** — Community
  Real-world skills examples from Andrej Karpathy.

---

## Rule Writing & Prompt Engineering

### Writing Effective Rules

- **[How to Write Rules for AI Agents](https://www.datablist.com/how-to/rules-writing-prompts-ai-agents)**  
  Framework for effective rule writing with 11 essential principles.

- **[.CursorRules Rules](https://dotcursorrules.com/)**  
  Collection of cursor rules examples from various projects and domains.

- **[Awesome Cursor Rules](https://apidog.com/blog/awesome-cursor-rules/)**  
  Curated list of effective rules with examples and explanations.

### Prompt Engineering

- **[Anthropic Prompt Engineering Guide](https://docs.anthropic.com/claude/docs/prompt-engineering)**  
  Official guide to crafting effective prompts for Claude models.

- **[OpenAI Prompt Engineering Guide](https://platform.openai.com/docs/guides/prompt-engineering)**  
  Best practices for prompting GPT models effectively.

---

## Tool Comparisons

- **[Cursor, Windsurf, Claude Code Compared](https://ai.trend.dmomo.co.kr/2025/06/cursor-windsurf-claude-code-compared.html)**  
  Feature-by-feature comparison of the three major AI coding tools.

- **[GitHub Rules for AI](https://github.com/hashiiiii/rules-for-ai)**  
  Cross-platform rules and guidelines repository.

---

## Academic and Research Papers

### Context Windows and Attention

- **[Lost in the Middle: How Language Models Use Long Contexts](https://arxiv.org/abs/2307.03172)**  
  Research on context rot and attention degradation in long contexts.

- **[The Curse of Recursion: Training on Generated Data](https://arxiv.org/abs/2305.17493)**  
  Implications for using AI-generated content in training and context.

### Prompt Engineering Research

- **[Large Language Models Are Human-Level Prompt Engineers](https://arxiv.org/abs/2211.01910)**  
  Research on automatic prompt optimization and engineering.

- **[Principled Instructions Are All You Need](https://arxiv.org/abs/2312.16171)**  
  Study on effective prompting principles across different models.

---

## Community Resources

### GitHub Repositories

- **[Awesome AI Coding](https://github.com/topics/ai-coding)**  
  Curated list of AI coding tools, resources, and examples.

- **[Context Engineering Examples](https://github.com/topics/context-engineering)**  
  Real-world examples of context engineering implementations.

### Forums and Communities

- **[Cursor Forum](https://forum.cursor.sh/)**  
  Official Cursor community forum for discussions and help.

- **[Anthropic Discord](https://discord.gg/anthropic)**  
  Official Discord for Claude and Claude Code discussions.

- **[r/cursor](https://reddit.com/r/cursor)**  
  Reddit community for Cursor IDE users.

- **[r/ClaudeAI](https://reddit.com/r/ClaudeAI)**  
  Reddit community for Claude AI discussions.

---

## Tools and Utilities

### Context Management

- **[OpenAI Tokenizer](https://platform.openai.com/tokenizer)**  
  Tool to count tokens and understand tokenization.

- **[Anthropic Console](https://console.anthropic.com/)**  
  Playground for testing Claude prompts and seeing token usage.

### Code Analysis

- **[GitHub Copilot](https://github.com/features/copilot)**  
  AI pair programmer integrated with VS Code and other IDEs.

- **[Tabnine](https://www.tabnine.com/)**  
  AI code completion supporting multiple IDEs.

---

## Related Documentation

See the following files in this repository for synthesized information:

| File | Content |
|------|---------|
| `docs/fundamentals/context-engineering.md` | Deep dive on context engineering principles |
| `docs/fundamentals/prompt-engineering.md` | Effective task prompts and patterns |
| `docs/fundamentals/memory-bank-pattern.md` | The key pattern for context management |
| `docs/tools/cursor-guide.md` | Complete Cursor configuration guide |
| `docs/tools/windsurf-guide.md` | Complete Windsurf configuration guide |
| `docs/tools/claude-code-guide.md` | Complete Claude Code configuration guide |
| `docs/tools/tool-comparison.md` | Side-by-side comparison of all three tools |
| `docs/best-practices/rule-writing.md` | How to write effective rules |
| `docs/best-practices/anti-patterns.md` | Common mistakes to avoid |
| `docs/best-practices/context-budgeting.md` | Managing context limits |
| `docs/workflow/spec-first-development.md` | Research → Plan → Implementation workflow |
| `docs/workflow/long-horizon-tasks.md` | Techniques for complex, long-running tasks |
| `docs/workflow/testing-context.md` | How to test your context configuration |

---

## Attribution

This repository synthesizes knowledge and patterns from:

- **FanDuel nexus** - Enterprise Spark library demonstrating memory-bank pattern at scale
- **Data API Platform** - FastAPI platform with task-based navigation and file size annotations
- **ai-presentation** - Context engineering and prompt engineering research compilation

---

## Contributing Resources

Found a great resource? Submit a PR adding it to the appropriate section with:
- Link and title
- Brief description (1-2 sentences)
- Why it's valuable

---

**Last Updated**: February 2026
