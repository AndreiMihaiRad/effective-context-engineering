# Testing Your Context Configuration

> **Key insight**: Context configuration should be tested like code. Measure effectiveness, iterate based on failures.

**Attribution**: Testing patterns derived from production context engineering implementations and real-world debugging.

---

## Why Test Context Configuration?

### The Problem

You can't know if your context setup is effective without testing it.

**Common failures**:
- AI misses documentation that exists
- AI makes assumptions instead of asking questions
- AI violates anti-patterns despite rules
- AI suggests outdated approaches

**Root cause**: Untested context configuration.

### The Solution

**Create a context test suite**:
- Specific test cases covering common scenarios
- Measurable success criteria
- Iterative improvement based on results

---

## Test Case Categories

### 1. Ambiguity Test

**Purpose**: Does AI ask questions when tasks are unclear?

**Good behavior**: Asks for clarification
**Bad behavior**: Makes assumptions

**Test Case 1: Vague Requirements**
```
Prompt: "Add caching to the API"

Expected:
- "Which endpoints should have caching?"
- "What caching backend (Redis, in-memory)?"
- "What TTL values?"

Failure modes:
- Implements Redis without asking (wrong assumption)
- Caches all endpoints (over-eager)
- Picks arbitrary TTL values
```

**Test Case 2: Missing Context**
```
Prompt: "Fix the login issue"

Expected:
- "What login issue are you experiencing?"
- "Which login method (OAuth, JWT)?"
- "What's the error or symptom?"

Failure modes:
- Assumes what the issue is
- Makes random changes
- "Fixed" something that wasn't broken
```

**How to Test**:
1. Give deliberately vague prompt
2. Check if AI asks clarifying questions
3. If it assumes instead → add rule: "Ask clarifying questions when task is ambiguous"

### 2. Recall Test

**Purpose**: Can AI retrieve information from early in context?

**Good behavior**: Recalls earlier information accurately
**Bad behavior**: Forgets or misremembers

**Test Case 1: Early Decision Recall**
```
Turn 1: "We decided to use Redis for caching with 1-hour TTL"
Turn 20: "Add caching to the new endpoint"

Expected:
- Uses Redis (not in-memory)
- Uses 1-hour TTL (not arbitrary value)

Failure modes:
- Implements in-memory caching (forgot decision)
- Uses 5-minute TTL (wrong recall)
```

**Test Case 2: File Location Recall**
```
Turn 5: "The auth logic is in src/auth/tokens.ts lines 45-67"
Turn 25: "Update the token validation"

Expected:
- Edits src/auth/tokens.ts lines 45-67
- Understands this is the right place

Failure modes:
- Creates new file (forgot location)
- Edits wrong section
- Searches for auth logic again
```

**How to Test**:
1. Provide information early in conversation
2. Many turns later, reference that information
3. Check if AI recalls correctly
4. If not → context window too polluted, consider reset

### 3. Tool Selection Test

**Purpose**: Does AI choose the right tool for the task?

**Good behavior**: Uses appropriate tool
**Bad behavior**: Uses wrong tool or asks for clarification

**Test Case 1: Finding Files**
```
Prompt: "Find all files containing 'OAuth'"

Expected tool:
- Grep/rg for content search

Wrong tools:
- SemanticSearch (for exact string, Grep is better)
- Read (too slow, searching blindly)
```

**Test Case 2: Understanding Concept**
```
Prompt: "How does our caching system work?"

Expected tool:
- Read memory-bank/caching/overview.md
- Or SemanticSearch "how caching works"

Wrong tools:
- Grep for "cache" (too many results)
- Reading random cache-related files
```

**How to Test**:
1. Give task that has obvious right tool
2. Check which tool AI uses
3. If wrong → rules unclear about when to use which tool

### 4. Degradation Test

**Purpose**: Does quality degrade in long sessions?

**Good behavior**: Consistent quality throughout
**Bad behavior**: Quality drops after many turns

**Test Case: 50-Turn Session**
```
Turns 1-10: Implement feature A
→ Measure quality (correctness, following patterns)

Turns 11-20: Implement feature B
→ Measure quality

Turns 21-30: Implement feature C
→ Measure quality

Turns 31-40: Implement feature D
→ Measure quality

Turns 41-50: Implement feature E
→ Measure quality

Expected: Consistent quality
Failure: Quality drops in turns 30+
```

**How to Test**:
1. Long conversation (50+ turns)
2. Track quality metrics at intervals
3. If degrading → Context reset needed more frequently

---

## Key Metrics

### 1. First-Attempt Success Rate

**Definition**: Percentage of tasks completed correctly on first try

**Measurement**:
```
First-Attempt Success Rate = (Correct on first try) / (Total tasks) × 100%
```

**Example**:
```
10 tasks given
7 correct on first try
3 required iteration

Success rate: 70%
```

**Target**: 80%+ for well-defined tasks

**What It Measures**:
- Is context sufficient?
- Are patterns clear?
- Are anti-patterns documented?

**Low success rate indicates**:
- Missing documentation
- Unclear rules
- Too much noise in context

### 2. Clarification Frequency

**Definition**: How often AI asks for clarification

**Measurement**:
```
Clarification Rate = (Tasks requiring clarification) / (Total tasks) × 100%
```

**Example**:
```
10 tasks given
3 asked for clarification
7 proceeded without questions

Clarification rate: 30%
```

**Target**:
- Vague tasks: 60%+ (should ask questions)
- Clear tasks: < 20% (should proceed)

**What It Measures**:
- Is AI appropriately cautious?
- Are tasks clear enough?

**Too high**: AI unsure of even clear tasks (rules too vague)
**Too low**: AI assumes instead of asking (add rule to ask questions)

### 3. Context Utilization

**Definition**: Percentage of context window used

**Measurement**:
```bash
# Claude Code
claude /cost
→ Shows token usage

# Cursor/Windsurf
[Check UI indicator]
```

**Target**: < 40% during implementation

**What It Measures**:
- Are rules appropriately sized?
- Is memory-bank being used correctly?
- Is context reset happening when needed?

**Too high**: Bloated rules, pre-loading docs, not resetting
**Too low**: May be okay, or under-utilizing available context

### 4. Token Efficiency

**Definition**: Results achieved per token spent

**Measurement**:
```
Token Efficiency = (Value delivered) / (Tokens consumed)
```

**Example**:
```
Task: Implement OAuth endpoint

Approach A:
- Pre-loaded all docs: 50,000 tokens
- Implementation: 30,000 tokens
- Total: 80,000 tokens

Approach B:
- Minimal rules: 500 tokens
- Read relevant docs: 3,000 tokens
- Implementation: 30,000 tokens
- Total: 33,500 tokens

Efficiency gain: 2.4x (same result, 58% fewer tokens)
```

**Target**: Maximize value per token

**What It Measures**:
- Is context being used efficiently?
- Are we loading irrelevant information?

---

## Creating a Test Suite

### Structure

```
tests/context/
├── ambiguity-tests.md
├── recall-tests.md
├── tool-selection-tests.md
├── degradation-tests.md
└── results.md
```

### Example Test File

**`tests/context/ambiguity-tests.md`**:

```markdown
# Ambiguity Test Cases

## Test 1: Vague Feature Request

**Prompt**: "Add caching to the API"

**Expected Behavior**:
- Asks which endpoints to cache
- Asks what caching backend to use
- Asks about TTL values

**Pass Criteria**:
- At least 2 clarifying questions asked
- No implementation without clarification

**Results**:
- Date: 2025-01-15
- Result: PASS - Asked about endpoints and TTL
- Notes: Assumed Redis (should ask), added rule to clarify backend

## Test 2: Missing Context

**Prompt**: "Fix the login issue"

**Expected Behavior**:
- Asks what issue is being experienced
- Asks which login method
- Asks for symptoms or error messages

**Pass Criteria**:
- At least 2 clarifying questions asked
- No blind fixing

**Results**:
- Date: 2025-01-15
- Result: FAIL - Made assumption about JWT
- Action: Add rule: "Never assume what 'issue' means - always ask"
```

### Running Tests

**Process**:
1. Start fresh conversation (reset context)
2. Run through test cases
3. Record pass/fail
4. Note failure patterns
5. Update rules/memory-bank based on failures
6. Re-run failed tests

**Frequency**:
- After major rule changes: Immediately
- Monthly: Full test suite
- Quarterly: Comprehensive review

---

## Iterative Improvement Process

### Step 1: Baseline

Run initial test suite:
```
Ambiguity tests: 60% pass (6/10)
Recall tests: 40% pass (4/10)
Tool selection: 70% pass (7/10)
Degradation: Not tested (need long session)

First-attempt success: 55%
Clarification rate: 25%
Context utilization: 65% (too high!)
```

**Issues identified**:
- AI makes assumptions (low ambiguity pass rate)
- AI forgets early context (low recall rate)
- Context bloated (65% utilization before starting)

### Step 2: Diagnose

**Ambiguity failures**: AI didn't ask questions when it should

**Root cause**: No rule about asking clarifying questions

**Fix**: Add rule:
```markdown
## Task Ambiguity

When a task request is vague or missing information:
- ASK clarifying questions
- DO NOT make assumptions
- Request specific requirements

Examples of vague tasks requiring clarification:
- "Add caching" → Ask which endpoints, what backend, what TTL
- "Fix the bug" → Ask what bug, what symptoms, where
- "Make it faster" → Ask what's slow, what's the target, what's measured
```

**Recall failures**: AI forgot decisions from earlier

**Root cause**: Context too polluted with irrelevant files

**Fix**: 
1. Reduce rule sizes (move details to memory-bank)
2. Add file size annotations to INDEX
3. Reset context more frequently

**Context bloat**: 65% before starting

**Root cause**: Rules too long, pre-loading docs

**Fix**:
1. Reduce workspace.mdc from 150 lines → 44 lines
2. Move details to memory-bank
3. Result: Utilization drops to 8%

### Step 3: Implement Fixes

Make changes to rules and memory-bank.

### Step 4: Re-Test

Run tests again:
```
Ambiguity tests: 90% pass (9/10) [+30%]
Recall tests: 80% pass (8/10) [+40%]
Tool selection: 80% pass (8/10) [+10%]
Degradation: 40+ turns, quality stable

First-attempt success: 75% [+20%]
Clarification rate: 40% [+15%]
Context utilization: 12% [-53%!]
```

**Result**: Significant improvement!

### Step 5: Iterate

Continue improving remaining failures.

---

## Common Test Failures and Fixes

### Failure: AI Doesn't Reference Memory-Bank

**Symptom**: AI asks questions instead of reading docs

**Test**:
```
Prompt: "How do I add a new transformation?"

Expected: Reads memory-bank/transformations/adding.md

Actual: Asks "What transformation do you want to add?"
```

**Fix**: Add rule:
```markdown
## Memory-Bank First

**Before asking questions**, check if memory-bank has the answer:
1. Read memory-bank/INDEX.md
2. Find relevant documentation
3. Read the documentation
4. THEN ask questions if still unclear
```

### Failure: AI Violates Anti-Patterns

**Symptom**: AI does things explicitly forbidden

**Test**:
```
Prompt: "Add a new transformation"

Expected: Checks 50+ existing transformations first

Actual: Creates new transformation without checking
```

**Fix**: Make anti-pattern more explicit:
```markdown
## Anti-Patterns

### Creating Transformations

❌ **NEVER** create a new transformation without first:
1. Reading `memory-bank/transformations/registry.md`
2. Verifying none of the 50+ existing transformations fit
3. If a similar one exists, extend it instead

✅ **ALWAYS** check existing transformations first
```

### Failure: Wrong Tool Selection

**Symptom**: AI uses semantic search for exact string

**Test**:
```
Prompt: "Find all files containing the exact string 'OAUTH_CLIENT_ID'"

Expected: Uses Grep

Actual: Uses SemanticSearch
```

**Fix**: Add tool guidance:
```markdown
## Tool Selection

### Finding Exact Strings
Use Grep/rg for exact string matches:
```bash
rg "OAUTH_CLIENT_ID"
```

### Finding by Concept
Use SemanticSearch for concepts:
"Where is OAuth configuration handled?"

### Finding by File Name
Use Glob for file names:
"*.config.ts"
```

---

## Test Automation (Future)

While currently manual, context testing could be automated:

**Automated Test Runner**:
```python
# tests/context/runner.py

def run_ambiguity_tests():
    for test in load_tests("ambiguity-tests.md"):
        response = ai.prompt(test.prompt)
        
        if has_clarifying_questions(response):
            record_pass(test)
        else:
            record_fail(test)
            
def run_recall_tests():
    conversation = ai.new_conversation()
    
    # Set up context
    conversation.send(test.setup_prompts)
    
    # Many turns later
    for i in range(20):
        conversation.send("Do something")
    
    # Test recall
    response = conversation.send(test.recall_prompt)
    
    if correctly_recalled(response, test.expected):
        record_pass(test)
    else:
        record_fail(test)
```

---

## Summary

### Testing Checklist

- [ ] Create test suite (ambiguity, recall, tool selection, degradation)
- [ ] Establish baseline metrics
- [ ] Run tests on fresh context
- [ ] Identify failure patterns
- [ ] Update rules/memory-bank based on failures
- [ ] Re-run tests to verify improvements
- [ ] Iterate monthly

### Key Metrics to Track

- [ ] First-attempt success rate (target: 80%+)
- [ ] Clarification frequency (appropriate to task clarity)
- [ ] Context utilization (target: < 40%)
- [ ] Token efficiency (maximize value per token)

### Continuous Improvement

- [ ] Test after rule changes
- [ ] Track metrics over time
- [ ] Review quarterly
- [ ] Remove obsolete rules
- [ ] Add rules for new failure patterns

---

## References

- See `docs/fundamentals/context-engineering.md` for core principles
- See `docs/best-practices/rule-writing.md` for writing effective rules
- See `docs/best-practices/anti-patterns.md` for common pitfalls
- See `examples/` for real-world test cases
