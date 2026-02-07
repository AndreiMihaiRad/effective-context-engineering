# GitHub Copilot Custom Instructions Template

This directory contains templates for configuring GitHub Copilot with the memory-bank pattern.

## Overview

GitHub Copilot supports repository-specific custom instructions through a special file:
- **Location**: `.github/copilot-instructions.md`
- **Format**: Plain Markdown (no frontmatter)
- **Size limit**: Recommended < 200 lines (no hard limit, but smaller is better)
- **Scope**: Repository-wide (all contributors)

## Quick Start

### 1. Create Directory Structure

```bash
mkdir -p .github
```

### 2. Copy Template

```bash
cp templates/copilot/.github/copilot-instructions.md.template .github/copilot-instructions.md
```

### 3. Customize

Edit `.github/copilot-instructions.md` and replace all placeholders:
- `[PROJECT_NAME]` - Your project name
- `[component-X]` - Your component/package names
- `[Language + version]` - Tech stack
- `[command]` - Common commands
- `[Pattern X]` - Key architectural patterns
- `[Anti-pattern X]` - Things to avoid
- `[Correct approach X]` - Recommended practices

### 4. Test

Open your repository in any IDE with GitHub Copilot installed. The instructions will be automatically loaded.

## File Structure

```
.github/
└── copilot-instructions.md    # Repository-wide custom instructions
```

## Key Differences from Other Tools

| Aspect | GitHub Copilot | Cursor | Windsurf | Claude Code |
|--------|----------------|--------|----------|-------------|
| **Location** | `.github/copilot-instructions.md` | `.cursor/rules/*.mdc` | `.windsurf/rules/*.md` | `CLAUDE.md` |
| **Format** | Plain Markdown | YAML + Markdown | Plain Markdown | Plain Markdown |
| **Multiple files** | No (single file) | Yes (multiple rules) | Yes (multiple rules) | Yes (hierarchy) |
| **Conditional loading** | No | Yes (globs) | Yes (globs) | Yes (directory-based) |
| **Size limit** | No hard limit | No hard limit | No hard limit | No hard limit |
| **Recommended size** | < 200 lines | < 100 lines per file | < 75 lines per file | < 300 lines |

## Best Practices

### 1. Keep It Minimal

GitHub Copilot loads the entire file for all sessions. Keep under 200 lines:
- ✅ Quick overview
- ✅ Critical constraints
- ✅ Memory-bank pointers
- ❌ Detailed documentation (use memory-bank/)
- ❌ Complete API references
- ❌ Exhaustive examples

### 2. Focus on Constraints, Not Documentation

**Bad** (documenting):
```markdown
## API Endpoints

### GET /users
Returns a list of all users.

Response schema:
{
  "users": [
    {"id": 1, "name": "John"}
  ]
}
```

**Good** (constraining):
```markdown
## API Constraints

- ❌ Never return sensitive fields (password_hash, tokens)
- ✅ Always paginate list endpoints (max 100 items)
- ✅ See `memory-bank/api-patterns.md` for endpoint patterns
```

### 3. Reference Memory-Bank

Instead of duplicating content, point to memory-bank:

```markdown
## Architecture

Brief overview here (3-4 sentences).

**Detailed documentation**:
- Architecture: `memory-bank/architecture/overview.md` (87 lines)
- API Patterns: `memory-bank/workflows/adding-endpoint.md` (365 lines)
- Testing: `memory-bank/workflows/testing.md` (360 lines)
```

### 4. Organize by Task, Not Structure

Help Copilot navigate by developer intent:

**Bad** (structural):
```markdown
## Components
- Component A
- Component B
- Component C
```

**Good** (task-based):
```markdown
## Quick Reference

### Adding New Feature
See `memory-bank/workflows/adding-feature.md`

### Debugging Issues
See `memory-bank/workflows/troubleshooting.md`

### Running Tests
See `memory-bank/workflows/testing.md`
```

### 5. Use Critical Constraints Effectively

Focus on what breaks builds, causes bugs, or violates security:

```markdown
## Critical Constraints

### Security (DO NOT)
- ❌ SQL string concatenation (allows injection)
- ❌ Hardcoded secrets
- ✅ Use parameterized queries
- ✅ Load secrets from environment/vault

### Performance (DO NOT)
- ❌ N+1 queries (use eager loading)
- ❌ Full table scans (add indexes)
- ✅ Batch database operations
```

## When to Use GitHub Copilot Instructions

### Good Use Cases
- ✅ Team repositories (all devs get same context)
- ✅ Consistent code generation needs
- ✅ Complementing other tools (can use with Cursor/Windsurf/Claude Code)
- ✅ Critical constraints that must never be violated

### Less Ideal Use Cases
- ❌ Personal preferences (use global settings)
- ❌ Extensive documentation (use memory-bank)
- ❌ File-specific rules (Copilot doesn't support conditional loading)

## Comparison with Other Tools

### GitHub Copilot
**Strengths**:
- Native GitHub integration
- Works across all IDEs (VS Code, JetBrains, etc.)
- Single file simplicity
- Team-wide consistency

**Limitations**:
- No conditional loading (single file always loaded)
- No multiple rule files
- Less control over when rules apply

**Best for**: Small to medium projects, team consistency

### Cursor
**Strengths**:
- Multiple rule files with conditional loading (globs)
- YAML frontmatter for metadata
- Fine-grained control

**Best for**: Large projects with many contexts

### Windsurf
**Strengths**:
- Multiple rule files with conditional loading
- Plain Markdown (simple)
- Similar to Cursor but simpler

**Best for**: Rapid prototyping, simplicity

### Claude Code
**Strengths**:
- File hierarchy (global → project → directory)
- Workflow-oriented
- Terminal integration

**Best for**: Terminal workflows, long-running tasks

## Multi-Tool Strategy

You can use GitHub Copilot instructions alongside other tools:

```
project/
├── .github/
│   └── copilot-instructions.md  # GitHub Copilot (200 lines)
├── .cursor/rules/
│   └── workspace.mdc             # Cursor (100 lines)
├── .windsurf/rules/
│   └── workspace.md              # Windsurf (75 lines)
├── CLAUDE.md                     # Claude Code (300 lines)
└── memory-bank/                  # Shared documentation
    ├── INDEX.md
    ├── architecture/
    ├── components/
    └── workflows/
```

**Benefits**:
- Team flexibility (use any tool)
- Content ~95% identical (95% reusable)
- Memory-bank 100% shared
- Single source of truth

## Examples

See the following examples for real-world implementations:
- **Simple project**: `examples/simple-project/`
- **FastAPI microservice**: `examples/fastapi-microservice/`
- **Monorepo platform**: `examples/monorepo-data-platform/`

## Testing Your Instructions

### 1. Verify File Location

```bash
ls -la .github/copilot-instructions.md
```

### 2. Test Code Generation

Open a file and trigger Copilot:
- Does it follow your naming conventions?
- Does it avoid anti-patterns?
- Does it reference memory-bank when appropriate?

### 3. Measure Effectiveness

Track over 2-3 weeks:
- First-attempt success rate (target: > 70%)
- Anti-pattern violations (target: < 5%)
- Correct pattern usage (target: > 80%)

## Troubleshooting

### Instructions Not Loading
- Verify file is at `.github/copilot-instructions.md`
- Check file is committed to repository
- Restart IDE/reload window
- Check GitHub Copilot extension is enabled

### Instructions Too Generic
- Add project-specific constraints
- Remove generic advice ("write clean code")
- Focus on patterns unique to your project

### Instructions Too Long
- Move documentation to memory-bank
- Keep only constraints and pointers
- Aim for < 200 lines

## Resources

- **Memory-Bank Pattern**: `docs/fundamentals/memory-bank-pattern.md`
- **Rule Writing**: `docs/best-practices/rule-writing.md`
- **Context Budgeting**: `docs/best-practices/context-budgeting.md`
- **Anti-Patterns**: `docs/best-practices/anti-patterns.md`

---

**Ready to get started?** Copy the template and customize for your project!
