# Context Health Check

Use this checklist to assess the health of your context engineering setup.

---

## Quick Health Indicators

### 🟢 Healthy Setup

- ✅ Prompts get good results on first attempt (> 70%)
- ✅ AI asks appropriate clarifying questions
- ✅ Context stays under 40% during work
- ✅ AI references memory-bank appropriately
- ✅ Long sessions maintain quality
- ✅ Team is satisfied with AI assistance

### 🟡 Needs Attention

- ⚠️ First-attempt success 50-70%
- ⚠️ Context hovers around 40-60%
- ⚠️ AI sometimes misses documentation
- ⚠️ Quality degrades in long sessions
- ⚠️ Some team members struggle

### 🔴 Unhealthy Setup

- ❌ First-attempt success < 50%
- ❌ Context regularly > 60%
- ❌ AI makes wrong assumptions
- ❌ AI ignores documented patterns
- ❌ Quality degrades quickly
- ❌ Team avoids using AI

---

## Detailed Health Checks

### 1. First-Attempt Success Rate

**Test**: Give AI 10 well-defined tasks, measure success

- [ ] **> 80% success** → 🟢 Excellent
- [ ] **70-80% success** → 🟢 Good
- [ ] **50-70% success** → 🟡 Needs improvement
- [ ] **< 50% success** → 🔴 Significant issues

**If poor**:
- [ ] Review rules - are they specific enough?
- [ ] Check memory-bank - is documentation clear?
- [ ] Test with clearer task prompts
- [ ] Add rules for observed failure patterns

---

### 2. Clarification Behavior

**Test**: Give vague and clear tasks, observe AI response

**For vague tasks** ("Add caching"):
- [ ] **AI asks questions** → 🟢 Good (should ask)
- [ ] **AI assumes without asking** → 🔴 Bad

**For clear tasks** ("Add Redis caching to /users endpoint, 1-hour TTL"):
- [ ] **AI proceeds without questions** → 🟢 Good
- [ ] **AI asks unnecessary questions** → 🟡 Rules may be unclear

**If wrong behavior**:
- [ ] Add rule: "Ask clarifying questions for vague tasks"
- [ ] Review rules for clarity on common tasks

---

### 3. Context Utilization

**Check**: Monitor context usage during typical work

**With Cursor/Windsurf**:
- [ ] Check token indicator in UI
- [ ] Before starting task, should be < 20%
- [ ] During work, should stay < 40%
- [ ] If > 60%, significant problem

**With Claude Code**:
```bash
claude /cost
```
- [ ] Before starting, should be < 20%
- [ ] During work, should stay < 40%
- [ ] If > 60%, rules are too large

**If context too high**:
- [ ] **Immediate**: Reduce rule file sizes
- [ ] Move documentation to memory-bank
- [ ] Remove unnecessary rules
- [ ] Target: Rules < 100 lines (Cursor), < 75 lines (Windsurf), < 150 lines (Claude)

---

### 4. Long-Session Quality

**Test**: Run 50+ turn conversation, check quality over time

- [ ] **Turns 1-20**: Quality check ___/10
- [ ] **Turns 21-40**: Quality check ___/10
- [ ] **Turns 41-50**: Quality check ___/10

**Results**:
- [ ] **Consistent quality** → 🟢 Good
- [ ] **Slight degradation after turn 40** → 🟡 Expected, use /clear
- [ ] **Significant degradation after turn 20** → 🔴 Context pollution

**If degrading too quickly**:
- [ ] Use `/clear` more frequently
- [ ] Review context budgeting
- [ ] Check for unnecessary file loading

---

### 5. Memory-Bank Usage

**Test**: "What documentation is available for [topic]?"

**Expected behavior**:
- [ ] AI references memory-bank/INDEX.md
- [ ] AI finds relevant documentation
- [ ] AI reads appropriate docs (not everything)

**Actual behavior**:
- [ ] **AI uses memory-bank correctly** → 🟢 Good
- [ ] **AI sometimes misses docs** → 🟡 Add more explicit rule
- [ ] **AI never uses memory-bank** → 🔴 Add "Memory-Bank First" rule

**If not using memory-bank**:
- [ ] Add explicit rule: "**ALWAYS consult** `memory-bank/INDEX.md` first"
- [ ] Test again with specific question

---

### 6. Pattern Adherence

**Test**: Ask AI to implement something with documented patterns

**Check**:
- [ ] **AI follows documented patterns** → 🟢 Good
- [ ] **AI sometimes deviates** → 🟡 Make patterns more explicit
- [ ] **AI ignores patterns** → 🔴 Patterns not clear enough

**Test cases**:
1. [ ] Ask to create new component → Uses documented pattern?
2. [ ] Ask to add feature → Follows architecture?
3. [ ] Ask to write tests → Uses correct framework/structure?

**If not following patterns**:
- [ ] Make patterns more explicit in rules
- [ ] Add code examples
- [ ] Add anti-patterns (what NOT to do)

---

### 7. Anti-Pattern Avoidance

**Test**: AI should not do things explicitly forbidden

**Check documented anti-patterns**:
- [ ] Anti-pattern 1: AI avoids it? ___
- [ ] Anti-pattern 2: AI avoids it? ___
- [ ] Anti-pattern 3: AI avoids it? ___

**Results**:
- [ ] **AI consistently avoids anti-patterns** → 🟢 Good
- [ ] **AI occasionally violates** → 🟡 Make anti-patterns clearer
- [ ] **AI frequently violates** → 🔴 Anti-patterns not explicit enough

**If violating anti-patterns**:
- [ ] Make anti-patterns more prominent
- [ ] Use "NEVER" keyword
- [ ] Add explanation of why it's wrong
- [ ] Add example of correct approach

---

### 8. File Size Awareness

**Test**: Ask for overview vs. detailed information

**Check**:
- [ ] For "quick overview" → AI reads lightweight docs
- [ ] For "detailed spec" → AI reads comprehensive docs
- [ ] AI doesn't pre-load all docs

**Results**:
- [ ] **AI chooses appropriate doc depth** → 🟢 Good
- [ ] **AI sometimes overloads** → 🟡 Reinforce file size annotations
- [ ] **AI always loads everything** → 🔴 Add file size guidance

**If poor file size awareness**:
- [ ] Ensure INDEX.md has file size categories
- [ ] Add usage strategy to INDEX.md
- [ ] Test with: "Give me a **quick** overview"

---

### 9. Context Clearing Frequency

**Check**: How often do team members clear context?

**Observations**:
- [ ] Clear between unrelated tasks: ___% of team
- [ ] Clear when context > 60%: ___% of team
- [ ] Never clear (accumulate): ___% of team

**Recommendations**:
- [ ] **Best practice**: Clear between unrelated tasks
- [ ] **Minimum**: Clear when context > 60%
- [ ] **If team never clears**: Add guidance to rules/CLAUDE.md

---

### 10. Team Satisfaction

**Survey team**:

1. **How satisfied are you with AI assistance?**
   - [ ] Very satisfied (9-10/10)
   - [ ] Satisfied (7-8/10)
   - [ ] Neutral (5-6/10)
   - [ ] Dissatisfied (< 5/10)

2. **How often do you use AI for coding tasks?**
   - [ ] Most tasks (> 80%)
   - [ ] Many tasks (50-80%)
   - [ ] Some tasks (20-50%)
   - [ ] Rarely (< 20%)

3. **Do AI suggestions match project patterns?**
   - [ ] Always/Usually
   - [ ] Sometimes
   - [ ] Rarely/Never

4. **Is documentation easy to find?**
   - [ ] Yes (memory-bank works well)
   - [ ] Sometimes
   - [ ] No (hard to find info)

**If low satisfaction**:
- [ ] Interview team members for specific issues
- [ ] Observe AI interactions
- [ ] Identify common failure patterns
- [ ] Update rules based on feedback

---

## Common Issues and Fixes

### Issue: Context Bloat

**Symptoms**:
- Context > 60% before starting work
- Slow AI responses
- Quality degradation

**Fixes**:
- [ ] Reduce rule file sizes (target < 100 lines)
- [ ] Move documentation to memory-bank
- [ ] Remove unnecessary rules
- [ ] More frequent context clearing

---

### Issue: AI Makes Wrong Assumptions

**Symptoms**:
- AI doesn't ask questions for vague tasks
- AI implements wrong patterns
- First-attempt success < 50%

**Fixes**:
- [ ] Add rule: "Ask clarifying questions for vague tasks"
- [ ] Make patterns more explicit
- [ ] Add more anti-patterns
- [ ] Test with ambiguous prompts

---

### Issue: AI Ignores Documentation

**Symptoms**:
- AI asks questions instead of reading docs
- AI implements patterns differently than documented
- Memory-bank not referenced

**Fixes**:
- [ ] Add explicit rule: "ALWAYS consult memory-bank/INDEX.md"
- [ ] Make memory-bank navigation more prominent
- [ ] Test: "What docs are available for X?"

---

### Issue: Quality Degrades in Long Sessions

**Symptoms**:
- Good quality for first 20 turns
- Degradation after 30-40 turns
- AI forgets earlier decisions

**Fixes**:
- [ ] Use `/clear` more frequently
- [ ] Implement compaction for very long tasks
- [ ] Structured note-taking for multi-session work

---

### Issue: Inconsistent Behavior

**Symptoms**:
- Works well sometimes, poorly other times
- Different team members get different results
- Unpredictable AI responses

**Fixes**:
- [ ] Make rules more specific (remove ambiguity)
- [ ] Add more concrete examples
- [ ] Standardize prompt patterns
- [ ] Share best practices across team

---

## Health Score

Calculate overall health score:

| Metric | Weight | Score (0-10) | Weighted |
|--------|--------|--------------|----------|
| First-attempt success | 25% | ___ | ___ |
| Clarification behavior | 10% | ___ | ___ |
| Context utilization | 20% | ___ | ___ |
| Long-session quality | 15% | ___ | ___ |
| Memory-bank usage | 10% | ___ | ___ |
| Pattern adherence | 10% | ___ | ___ |
| Anti-pattern avoidance | 5% | ___ | ___ |
| Team satisfaction | 5% | ___ | ___ |
| **TOTAL** | **100%** | | **___** |

**Interpretation**:
- **8.0-10.0**: 🟢 Excellent - Minor tweaks only
- **6.0-7.9**: 🟢 Good - Some improvements needed
- **4.0-5.9**: 🟡 Fair - Significant improvements needed
- **< 4.0**: 🔴 Poor - Major overhaul needed

---

## Action Plan

Based on health check, create action plan:

### High Priority (This Week)
- [ ] [Issue found]
- [ ] [Issue found]
- [ ] [Issue found]

### Medium Priority (This Month)
- [ ] [Issue found]
- [ ] [Issue found]

### Low Priority (Next Quarter)
- [ ] [Improvement idea]
- [ ] [Improvement idea]

---

## Review Schedule

- **Weekly**: Quick health indicators
- **Monthly**: Full health check
- **Quarterly**: Comprehensive review + team survey

---

## References

- **Setup guide**: `checklists/setup-checklist.md`
- **Rule review**: `checklists/rule-review-checklist.md`
- **Testing context**: `docs/workflow/testing-context.md`
- **Context budgeting**: `docs/best-practices/context-budgeting.md`
