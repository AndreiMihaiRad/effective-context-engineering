# Context Engineering Setup Checklist

Use this checklist when setting up context engineering for a new project.

---

## Phase 1: Choose Your Tools

- [ ] **Decide which AI tool(s) to use**
  - [ ] Cursor (IDE-based, precise control with globs)
  - [ ] Windsurf (IDE-based, simple setup)
  - [ ] Claude Code (terminal-based, workflow-focused)
  - [ ] Multiple tools (recommended for team flexibility)

---

## Phase 2: Create Directory Structure

- [ ] **Create rules directory for your tool(s)**
  - [ ] Cursor: `mkdir -p .cursor/rules`
  - [ ] Windsurf: `mkdir -p .windsurf/rules`
  - [ ] Claude Code: (CLAUDE.md in project root)

- [ ] **Create memory-bank structure**
  - [ ] `mkdir -p memory-bank/architecture`
  - [ ] `mkdir -p memory-bank/components`
  - [ ] `mkdir -p memory-bank/workflows`
  - [ ] `mkdir -p memory-bank/tech-stack` (optional)

---

## Phase 3: Create memory-bank/INDEX.md

- [ ] **Copy INDEX template**
  - [ ] `cp templates/memory-bank/INDEX.md.template memory-bank/INDEX.md`

- [ ] **Fill in placeholders**
  - [ ] Add 3-5 most common tasks to "Navigation by Task"
  - [ ] Create documentation files for those tasks
  - [ ] Count lines and add to file size reference
  - [ ] Update directory structure tree
  - [ ] Add usage strategy

- [ ] **Target**: INDEX.md under 100 lines

---

## Phase 4: Write Workspace-Level Rules

### For Cursor

- [ ] **Create `.cursor/rules/workspace.mdc`**
  - [ ] Add YAML frontmatter with `alwaysApply: true`
  - [ ] Add workspace overview with package table
  - [ ] Add memory-bank navigation pointer
  - [ ] Add code standards (formatting, type hints, naming)
  - [ ] Target < 100 lines

### For Windsurf

- [ ] **Create `.windsurf/rules/workspace.md`**
  - [ ] Add workspace overview (no frontmatter)
  - [ ] Add memory-bank navigation pointer
  - [ ] Add code standards
  - [ ] Target < 75 lines (all rules load for every session)

### For Claude Code

- [ ] **Create `CLAUDE.md` in project root**
  - [ ] Start from `templates/claude/CLAUDE.md.template` (not `/init`)
  - [ ] Use WHY/WHAT/HOW framework
  - [ ] Place critical constraints at beginning AND end
  - [ ] Add workspace overview
  - [ ] Add development commands
  - [ ] Add architecture summary
  - [ ] Add memory-bank navigation
  - [ ] Add code standards
  - [ ] Target < 150 lines

- [ ] **Create `.claude/settings.json`** (optional but recommended)
  - [ ] Copy from `templates/claude/settings.json.template`
  - [ ] Configure `allow` permissions for safe commands
  - [ ] Configure `deny` for dangerous operations
  - [ ] Add hooks if using linters (PostToolUse)

- [ ] **Optionally create skills/commands/agents**
  - [ ] Skills in `.claude/skills/` for AI-discovered capabilities
  - [ ] Commands in `.claude/commands/` for user-invoked workflows
  - [ ] Agents in `.claude/agents/` for autonomous sub-tasks
  - [ ] Rules in `.claude/rules/` for path-scoped modular rules

---

## Phase 5: Write Project-Level Rules (if needed)

Skip this if you have a single project. For monorepos or multi-project repos:

### For Cursor

- [ ] **Create `.cursor/rules/[project].mdc` for each project**
  - [ ] Add YAML frontmatter with `globs: ["project/**/*"]`
  - [ ] Add project context
  - [ ] Add key patterns (3-5)
  - [ ] Add memory-bank reference for project
  - [ ] Add development commands
  - [ ] Add anti-patterns (3-5)
  - [ ] Target < 100 lines per project

### For Windsurf

- [ ] **Create `.windsurf/rules/[project].md` for each project**
  - [ ] Add scope note (which files this applies to)
  - [ ] Add project context
  - [ ] Add key patterns
  - [ ] Add memory-bank reference
  - [ ] Add development commands
  - [ ] Add anti-patterns
  - [ ] Target < 75 lines per project
  - [ ] Maximum 3-4 project files total

### For Claude Code

- [ ] **Optionally create subdirectory CLAUDE.md files**
  - [ ] Only if project needs directory-specific overrides
  - [ ] Keep minimal (overrides only)

---

## Phase 6: Document Key Patterns and Anti-Patterns

- [ ] **Identify 3-5 most important patterns**
  - [ ] Architectural patterns (e.g., "Factory pattern for readers")
  - [ ] Code organization patterns
  - [ ] Naming patterns

- [ ] **Identify 3-5 most dangerous anti-patterns**
  - [ ] Things that break the system
  - [ ] Common mistakes to avoid
  - [ ] Be specific, not generic

- [ ] **Add to rules files** (not memory-bank)
  - [ ] Rules = constraints
  - [ ] Memory-bank = documentation

---

## Phase 7: Add Development Commands

- [ ] **Document common commands**
  - [ ] How to run tests
  - [ ] How to format code
  - [ ] How to run linters
  - [ ] How to start dev server
  - [ ] How to build/deploy

- [ ] **Add to appropriate location**
  - [ ] Cursor/Windsurf: In project rules
  - [ ] Claude Code: In CLAUDE.md "Development Commands" section

---

## Phase 8: Create Initial Memory-Bank Documentation

Start minimal, add based on needs:

- [ ] **Create architecture overview** (`memory-bank/architecture/overview.md`)
  - [ ] High-level system design (< 150 lines)
  - [ ] Key components
  - [ ] Data flow

- [ ] **Create 2-3 workflow guides** (`memory-bank/workflows/`)
  - [ ] Most common task #1 (e.g., adding feature)
  - [ ] Most common task #2 (e.g., testing)
  - [ ] Most common task #3 (e.g., deployment)
  - [ ] Target 150-350 lines each

- [ ] **Count lines and update INDEX.md**
  - [ ] `wc -l memory-bank/**/*.md`
  - [ ] Add to file size reference

---

## Phase 9: Test with a Simple Task

- [ ] **Choose a simple, common task**
  - [ ] Example: "Add a new API endpoint"
  - [ ] Example: "Write tests for X component"

- [ ] **Run the task with AI**
  - [ ] Start fresh context
  - [ ] Give task prompt
  - [ ] Observe AI behavior

- [ ] **Check for success**
  - [ ] Did AI reference memory-bank?
  - [ ] Did AI follow patterns?
  - [ ] Did AI avoid anti-patterns?
  - [ ] Was first attempt successful?

---

## Phase 10: Iterate Based on Failures

- [ ] **Observe AI behavior for a week**
  - [ ] Note when AI makes wrong assumptions
  - [ ] Note when AI violates patterns
  - [ ] Note when AI asks questions vs. checking docs

- [ ] **Add rules for observed failures**
  - [ ] Specific, not generic
  - [ ] Actionable constraints
  - [ ] Reference memory-bank for details

- [ ] **Don't add rules for one-off issues**
  - [ ] Wait for patterns to emerge
  - [ ] Add rules only for repeated failures

---

## Final Checks

- [ ] **Rule sizes are appropriate**
  - [ ] Cursor: Each rule < 100 lines
  - [ ] Windsurf: Each rule < 75 lines, max 3-4 files
  - [ ] Claude Code: CLAUDE.md < 150 lines

- [ ] **Memory-bank INDEX < 100 lines**
  - [ ] If larger, move content to docs

- [ ] **All files are version controlled**
  - [ ] Commit `.cursor/rules/`, `.windsurf/rules/`, `CLAUDE.md`
  - [ ] Commit `memory-bank/`
  - [ ] Don't commit user-level config

- [ ] **Documentation references are working**
  - [ ] Test: "What documentation is available for X?"
  - [ ] AI should reference memory-bank INDEX

- [ ] **Context utilization is reasonable**
  - [ ] Check with `/cost` (Claude Code) or UI indicator
  - [ ] Target < 40% during implementation
  - [ ] If higher, reduce rule sizes

---

## Success Criteria

Your context engineering setup is ready when:

- ✅ **First-attempt success rate > 70%** for well-defined tasks
- ✅ **AI references memory-bank** appropriately
- ✅ **AI follows documented patterns**
- ✅ **AI avoids documented anti-patterns**
- ✅ **Context utilization < 40%** during work
- ✅ **Rules are minimal** (Cursor < 100 lines, Windsurf < 75 lines, Claude < 150 lines)
- ✅ **Team members can onboard** using the setup in < 1 hour

---

## Maintenance Schedule

- **Weekly**: Observe AI behavior, note failure patterns
- **Monthly**: Review and update rules based on observations
- **Quarterly**: Comprehensive review
  - Remove obsolete rules
  - Update for architecture changes
  - Verify memory-bank accuracy
  - Test context effectiveness

---

## Next Steps After Setup

1. **Share with team**
   - Commit configuration to repository
   - Document in project README
   - Onboard team members

2. **Create test suite** (optional but recommended)
   - See `docs/workflow/testing-context.md`
   - Create ambiguity tests
   - Create recall tests
   - Track metrics over time

3. **Expand memory-bank** as needed
   - Add documentation for complex components
   - Add troubleshooting guides
   - Add architecture decision records

4. **Consider multi-tool support**
   - If using only Cursor, add Windsurf rules (easy conversion)
   - If using only Claude Code, add Cursor rules for IDE users
   - Memory-bank is shared across all tools

---

## References

- **Getting started guides**: `docs/tools/cursor-guide.md`, `windsurf-guide.md`, `claude-code-guide.md`
- **Memory-bank pattern**: `docs/fundamentals/memory-bank-pattern.md`
- **Rule writing**: `docs/best-practices/rule-writing.md`
- **Templates**: `templates/` directory
- **Examples**: `examples/` directory
