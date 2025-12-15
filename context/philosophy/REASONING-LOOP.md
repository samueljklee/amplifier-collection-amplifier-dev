# Reasoning Loop

> Step-by-step workflow from problem to validated solution

---

## The Core Loop

```
┌─────────────────────────────────────────────────────────────────┐
│                    START: Receive Task/Error                     │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  1. CLARIFY: What exactly is the problem/goal?                  │
│     - If error: extract exact message, trace to source          │
│     - If feature: understand what "done" looks like             │
│     - If fuzzy: ask clarifying questions                        │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  2. SEARCH: Find relevant code and context                      │
│     - Grep for error strings, function names, patterns          │
│     - Read related files to understand structure                 │
│     - Check sibling projects for reference implementations      │
└─────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  3. HYPOTHESIZE: Form a theory about root cause or approach     │
│     - State the hypothesis explicitly                           │
│     - If multiple options: compare in table format              │
└─────────────────────────────────────────────────────────────────┘
                                │
                    ┌───────────┴───────────┐
                    │  Confidence check?     │
                    │  (Human asks or        │
                    │   high-stakes change)  │
                    └───────────┬───────────┘
                          YES   │   NO
                    ┌───────────┘   └───────────┐
                    ▼                           ▼
┌──────────────────────────────┐    ┌──────────────────────────────┐
│  3a. VERIFY HYPOTHESIS       │    │  4. IMPLEMENT                 │
│  - Search for counter-       │    │  - Make smallest change that  │
│    evidence                  │    │    could work                 │
│  - Check docs/specs          │    │  - Follow existing patterns   │
│  - Cite multiple evidence    │    │  - Update tests if needed     │
│    sources                   │    │                               │
└──────────────────────────────┘    └──────────────────────────────┘
                    │                           │
                    └───────────┬───────────────┘
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│  5. VALIDATE: Confirm the change works                          │
│     - Run tests (or create standalone validation if blocked)    │
│     - Grep for old patterns to ensure complete removal          │
│     - Check related areas for collateral issues                 │
└─────────────────────────────────────────────────────────────────┘
                                │
                    ┌───────────┴───────────┐
                    │  Still broken?         │
                    └───────────┬───────────┘
                          YES   │   NO
                    ┌───────────┘   └───────────┐
                    ▼                           ▼
┌──────────────────────────────┐    ┌──────────────────────────────┐
│  5a. WIDEN SEARCH            │    │  6. DOCUMENT & CLOSE         │
│  - Look for same pattern     │    │  - Summarize what changed    │
│    elsewhere                 │    │  - Update README/docs        │
│  - Check if tests enforce    │    │  - Confirm with human        │
│    old behavior              │    │                               │
│  - Return to step 2          │    │                               │
└──────────────────────────────┘    └──────────────────────────────┘
```

---

## Conditional Branches

| Condition | Action |
|-----------|--------|
| **Requirements fuzzy** | Ask clarifying questions before searching |
| **Multiple valid approaches** | Create comparison table; prefer what existing code does |
| **Fix didn't work** | Widen search to tests, sibling files, version issues |
| **Dependencies block testing** | Create standalone validation that avoids dependencies |
| **High-stakes change** | Gather multiple evidence sources before implementing |
| **Confidence challenged** | Pause, gather evidence, cite sources, then proceed |

---

## Phase Details

### Phase 1: Clarify

**For bugs:**
- Extract the exact error message
- Identify the file/line where it occurs
- Understand what behavior is expected vs actual

**For features:**
- Define what "done" looks like
- Identify constraints (what can't change)
- Clarify success criteria

**Signals you need more clarity:**
- You're not sure where to start searching
- Multiple interpretations seem valid
- The scope feels unbounded

### Phase 2: Search

**Primary searches:**
- Grep for error strings
- Grep for function/class names mentioned
- Read files that seem related

**Secondary searches:**
- Check sibling projects for reference implementations
- Search for similar patterns in the codebase
- Look at recent changes to related files

**Search order:**
1. Direct matches (error string, function name)
2. Related files (same directory, similar names)
3. Sibling projects (how do others do this?)

### Phase 3: Hypothesize

**Good hypothesis format:**
> "The error occurs because X is doing Y, which Z doesn't support."

**When multiple approaches exist:**

| Approach | Pros | Cons | Used By |
|----------|------|------|---------|
| A | ... | ... | Project X, Y |
| B | ... | ... | Project Z |

**Default:** Follow the majority unless you have specific evidence they're wrong.

### Phase 4: Implement

**Principles:**
- Make the smallest change that could work
- Follow existing patterns in the codebase
- One change, one validation (don't bundle)

**Check before committing:**
- Are there tests that assert the old behavior?
- Does this match how sibling projects do it?
- Is there duplicate code that also needs changing?

### Phase 5: Validate

**Multi-channel verification:**

| Channel | Command/Action |
|---------|---------------|
| Run tests | `pytest`, `npm test`, etc. |
| Grep for old pattern | `grep -r "old_pattern"` |
| Check related areas | Read files that import/use changed code |
| Version check | Verify installed versions match expected |

**If validation fails:**
- Don't repeat the same fix
- Widen the search (tests, configs, sibling files)
- Ask "what else could cause this?"

### Phase 6: Document & Close

**Summary format:**
```
Root Cause: [What was wrong]
Solution: [What was changed]
Files Modified: [List]
Validation: [How it was verified]
```

**Update docs if:**
- Behavior changed
- Configuration options added
- New patterns introduced

---

## JSON Representation

For tooling integration:

```json
{
  "reasoning_loop": {
    "nodes": [
      {"id": "clarify", "name": "Clarify Problem/Goal"},
      {"id": "search", "name": "Search for Context"},
      {"id": "hypothesize", "name": "Form Hypothesis"},
      {"id": "verify", "name": "Verify Hypothesis", "trigger": "confidence_check OR high_stakes"},
      {"id": "implement", "name": "Implement Change"},
      {"id": "validate", "name": "Validate Change"},
      {"id": "widen_search", "name": "Widen Search", "trigger": "validation_failed"},
      {"id": "document", "name": "Document & Close"}
    ],
    "edges": [
      {"from": "clarify", "to": "search", "condition": "always"},
      {"from": "search", "to": "hypothesize", "condition": "always"},
      {"from": "hypothesize", "to": "verify", "condition": "confidence_check OR high_stakes"},
      {"from": "hypothesize", "to": "implement", "condition": "confident"},
      {"from": "verify", "to": "implement", "condition": "evidence_supports"},
      {"from": "implement", "to": "validate", "condition": "always"},
      {"from": "validate", "to": "widen_search", "condition": "still_broken"},
      {"from": "validate", "to": "document", "condition": "working"},
      {"from": "widen_search", "to": "search", "condition": "always"}
    ]
  }
}
```
