# Rule Review Checklist

Use this checklist when reviewing your context engineering rules for quality and effectiveness.

---

## Size and Organization

- [ ] **Each rule file is appropriately sized**
  - [ ] Cursor rules: < 100 lines each
  - [ ] Windsurf rules: < 75 lines each
  - [ ] Claude CLAUDE.md: < 300 lines
  - [ ] If too large, split or move to memory-bank

- [ ] **Rules are split by concern, not arbitrarily**
  - [ ] Each file has a clear, focused purpose
  - [ ] No "rules-part1.md" or "rules-part2.md"
  - [ ] Logical groupings (workspace, frontend, backend, etc.)

- [ ] **Total number of rule files is reasonable**
  - [ ] Cursor: 3-7 files typical
  - [ ] Windsurf: 3-4 files maximum
  - [ ] Claude Code: 1-3 CLAUDE.md files (project + subdirs)

---

## Content Quality

- [ ] **Rules are specific, not generic**
  - [ ] ❌ "Write clean code", "Follow best practices"
  - [ ] ✅ "Never override run() in BaseJob"
  - [ ] ✅ "Use parameterized queries, not string concatenation"

- [ ] **Anti-patterns are concrete and actionable**
  - [ ] Each anti-pattern includes:
    - [ ] What not to do (specific)
    - [ ] Why it's wrong
    - [ ] What to do instead

- [ ] **Patterns include examples**
  - [ ] Code snippets where helpful
  - [ ] Clear before/after comparisons
  - [ ] Real project examples

- [ ] **No obvious or trivial advice**
  - [ ] Remove language basics (AI already knows)
  - [ ] Remove generic programming wisdom
  - [ ] Keep only project-specific constraints

---

## Memory-Bank Integration

- [ ] **Rules reference memory-bank for details**
  - [ ] "See `memory-bank/X/` for ..." appears in rules
  - [ ] Rules don't duplicate memory-bank content
  - [ ] Clear pointers to where to find more info

- [ ] **Documentation is in memory-bank, not rules**
  - [ ] Rules = constraints ("what"/"what not")
  - [ ] Memory-bank = documentation ("how"/"why")
  - [ ] No > 50 line explanations in rules

- [ ] **Memory-bank INDEX is referenced prominently**
  - [ ] "Start with `memory-bank/INDEX.md`" in rules
  - [ ] Clear section header for memory-bank navigation

---

## Specificity and Actionability

- [ ] **Constraints are clear and unambiguous**
  - [ ] ✅ "Max 3 parameters per function"
  - [ ] ❌ "Keep functions small"

- [ ] **Requirements are measurable**
  - [ ] ✅ "80%+ test coverage for new code"
  - [ ] ❌ "Write good tests"

- [ ] **Tech stack is specific**
  - [ ] ✅ "Python 3.12+", "React 18", "pytest"
  - [ ] ❌ "Python", "React", "testing framework"

- [ ] **Commands include examples**
  - [ ] Not just "run tests"
  - [ ] But "pytest tests/unit/ -v"

---

## Duplication

- [ ] **No content duplication across rule files**
  - [ ] Same pattern not described in multiple files
  - [ ] Shared content moved to workspace rule
  - [ ] Project rules only have project-specific content

- [ ] **No duplication between rules and memory-bank**
  - [ ] Long explanations in memory-bank only
  - [ ] Rules reference memory-bank
  - [ ] Single source of truth

- [ ] **No duplication across tools**
  - [ ] Cursor and Windsurf rules ~95% identical
  - [ ] Differences only in format (YAML vs Markdown)
  - [ ] Claude CLAUDE.md may include commands not in others

---

## Tool-Specific Concerns

### Cursor Rules

- [ ] **Glob patterns are appropriate**
  - [ ] Specific enough to target intended files
  - [ ] Not overly broad (`**/*` is usually wrong)
  - [ ] Test: Edit file, verify correct rules load

- [ ] **alwaysApply used correctly**
  - [ ] Only for workspace-level rules
  - [ ] Typically 1 file with alwaysApply
  - [ ] Other files use globs

- [ ] **Description fields are clear**
  - [ ] Describes what the rule covers
  - [ ] < 100 characters
  - [ ] Shows in Cursor UI

### Windsurf Rules

- [ ] **Files include scope notes**
  - [ ] Since no globs, scope stated explicitly
  - [ ] "**Scope**: `backend/**/*.py`"
  - [ ] Clear when this rule applies

- [ ] **Extra concise (all load always)**
  - [ ] Target < 75 lines (more strict than Cursor)
  - [ ] Maximum 3-4 files total
  - [ ] Even more reliance on memory-bank

- [ ] **No YAML frontmatter**
  - [ ] Plain Markdown only
  - [ ] Content otherwise identical to Cursor

### Claude Code

- [ ] **Development commands included**
  - [ ] How to run tests
  - [ ] How to start dev server
  - [ ] How to build/deploy
  - [ ] With concrete examples

- [ ] **Quick reference section useful**
  - [ ] Most common tasks
  - [ ] Brief examples or pointers
  - [ ] Not duplicating full workflows

- [ ] **Critical constraints highlighted**
  - [ ] Security section with ❌/✅
  - [ ] Performance section with ❌/✅
  - [ ] Breaking changes section

---

## Testing and Validation

- [ ] **Rules have been tested with real tasks**
  - [ ] Give AI a common task
  - [ ] Verify it follows patterns
  - [ ] Verify it avoids anti-patterns

- [ ] **AI references memory-bank appropriately**
  - [ ] Test: "What documentation is available?"
  - [ ] AI should mention memory-bank/INDEX.md

- [ ] **First-attempt success rate is good**
  - [ ] > 70% for well-defined tasks
  - [ ] AI asks clarifying questions when appropriate
  - [ ] AI doesn't make wrong assumptions

- [ ] **Context utilization is healthy**
  - [ ] < 40% during implementation
  - [ ] If higher, rules may be too large

---

## Consistency Across Tools

If supporting multiple tools:

- [ ] **Content is ~95% identical**
  - [ ] Same patterns documented
  - [ ] Same anti-patterns listed
  - [ ] Same memory-bank references

- [ ] **Differences are format-only**
  - [ ] Cursor: YAML frontmatter
  - [ ] Windsurf: Scope notes
  - [ ] Claude: Development commands

- [ ] **Memory-bank is shared**
  - [ ] Same memory-bank/ directory
  - [ ] Same INDEX.md
  - [ ] No tool-specific forks

---

## Version Control

- [ ] **Rules are committed to repository**
  - [ ] `.cursor/rules/*.mdc`
  - [ ] `.windsurf/rules/*.md`
  - [ ] `CLAUDE.md`
  - [ ] `memory-bank/`

- [ ] **User-level config is NOT committed**
  - [ ] `~/.cursor/` (personal config)
  - [ ] `~/.claude/CLAUDE.md` (global preferences)

- [ ] **Changes are reviewed in PRs**
  - [ ] Rule changes visible to team
  - [ ] Rationale documented in commit messages

---

## Maintenance

- [ ] **Last review date is recent**
  - [ ] Monthly reviews scheduled
  - [ ] Quarterly comprehensive reviews

- [ ] **Obsolete rules removed**
  - [ ] Old patterns no longer in use
  - [ ] References to deprecated tools
  - [ ] Outdated anti-patterns

- [ ] **Rules updated for architecture changes**
  - [ ] New patterns added
  - [ ] Old patterns removed/updated
  - [ ] Memory-bank references current

---

## Red Flags

Watch for these warning signs:

- [ ] ⚠️ **Any rule file > 150 lines** → Split or move to memory-bank
- [ ] ⚠️ **Generic advice** → Remove or make specific
- [ ] ⚠️ **Duplication** → Create single source of truth
- [ ] ⚠️ **No memory-bank references** → Add pointers
- [ ] ⚠️ **Complex explanations in rules** → Move to memory-bank
- [ ] ⚠️ **Context > 50% before starting work** → Rules too large
- [ ] ⚠️ **AI not following rules** → Rules unclear or too vague
- [ ] ⚠️ **AI asking instead of reading docs** → Add "check memory-bank first" rule

---

## Action Items

After review, create action items for issues found:

- [ ] **High Priority** (do immediately)
  - [ ] Rules > 150 lines → Split/move
  - [ ] Context bloat → Reduce sizes
  - [ ] AI not following rules → Clarify

- [ ] **Medium Priority** (this month)
  - [ ] Generic advice → Make specific
  - [ ] Duplication → Consolidate
  - [ ] Missing memory-bank refs → Add

- [ ] **Low Priority** (next quarter)
  - [ ] Formatting improvements
  - [ ] Additional examples
  - [ ] Documentation enhancements

---

## Review Schedule

- **Weekly**: Observe AI behavior, note issues
- **Monthly**: Run this checklist, fix high/medium priority issues
- **Quarterly**: Comprehensive review + cleanup

---

## Success Criteria

Your rules are in good shape when:

- ✅ All checklist items pass
- ✅ No red flags present
- ✅ AI behavior is consistently good
- ✅ Team is satisfied with AI assistance
- ✅ Context utilization is healthy
- ✅ Rules are being maintained

---

## References

- **Rule writing guide**: `docs/best-practices/rule-writing.md`
- **Anti-patterns guide**: `docs/best-practices/anti-patterns.md`
- **Memory-bank pattern**: `docs/fundamentals/memory-bank-pattern.md`
- **Context budgeting**: `docs/best-practices/context-budgeting.md`
