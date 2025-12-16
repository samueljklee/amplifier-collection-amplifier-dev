# Velocity Patterns

> Fast-execution patterns for iterative shipping when territory is familiar

---

## Overview

These patterns complement the deep-reasoning patterns in PATTERNS.md. Use **velocity mode** when:
- You're in familiar territory (known codebase, clear goals)
- Stakes per action are low (easy rollback)
- Momentum matters more than perfection
- Iteration is cheap (git commits, quick tests)

Use **deep mode** (PATTERNS.md) when:
- Debugging complex/unclear problems
- Stakes are high (production, breaking changes)
- You need to understand before acting

---

## Velocity Patterns

### 1. Verified Commit Cycling

**Description:** Break work into small commits, but only after verification.

**How it works:**
- Each logical change gets its own commit
- **Before committing**: Build passes, tests pass, behavior verified
- Commit messages describe the "what" and "why"
- Push after verification to create rollback points

**The Commit Gate:**
```
Code → Build → Test → Verify → (User says "commit") → Commit
```

**Example:** Instead of one big "Add feature X" commit, create:
- `feat: Add X component skeleton` (after build passes)
- `feat: Wire X to API` (after integration test passes)
- `fix: Handle edge case in X` (after edge case verified)
- `docs: Update README for X` (after docs reviewed)

**When to use:** Shipping features, iterating on known patterns
**When to avoid:** Exploratory work where you might revert everything

**Critical:** Never commit untested code. A commit implies "this works."

---

### 2. Directive Delegation

**Description:** Issue terse, action-oriented commands trusting the executor to fill in details.

**How it works:**
- State the outcome, not the steps
- Trust context from prior exchanges
- Correct quickly if results diverge

**Examples:**
- "commit the staged changes with a proper message"
- "push it"
- "do those too"
- "there's more changes, do the same"

**When to use:** Trust is established, stakes are low, patterns are clear
**When to avoid:** Novel territory, high stakes, ambiguous requirements

**Risk mitigation:** Review outputs before they become permanent (commits, pushes)

---

### 3. Concrete Evidence Debugging

**Description:** When something fails, lead with logs/output rather than abstract descriptions.

**How it works:**
- Paste exact error messages
- Include console output
- Show actual vs. expected (when known)

**Example:**
```
MountPlanBuilder.tsx:502 Loaded agent content, length: 0
Agent has no content: bug-hunter
```

Better than: "The agent loading isn't working"

**Why it helps:** Gives the executor actionable data immediately

---

### 4. Simplification Questioning

**Description:** After seeing a solution, challenge unnecessary complexity.

**Key questions:**
- "Do we need X?"
- "Is Y sufficient?"
- "Can't the caller just..."
- "What if we removed..."

**Example:** "do we need the convention search? is module id sufficient? and it's the caller's responsibility to pass the right module id"

**Outcome:** Keeps solutions minimal and maintainable

---

### 5. Validation Loop Request

**Description:** After a fix, request to test in real environment before committing.

**Pattern:**
1. Implement change
2. "bring that change into [test env] so I can validate"
3. Test manually
4. Proceed or iterate

**When to use:** Non-trivial changes, integration points
**When to skip:** Trivial formatting, obvious fixes

---

### 6. Gap Identification Prompt

**Description:** After understanding a system, explicitly ask for overlooked issues.

**Key phrases:**
- "Do you see any gaps?"
- "What am I missing?"
- "What could go wrong?"

**Example:** "do you see any gaps for the web UI showing mount plan json in dependency graph dialog?"

**Why it works:** Leverages broader perspective to catch blind spots

---

### 7. Architecture Trace Request

**Description:** When confused about behavior, trace data flow before attempting fixes.

**Pattern:**
- "I want to understand how X gets passed into Y"
- "Show me the flow from [input] to [output]"
- "How does this repo use [component]?"

**When to use:** Before fixing, when behavior is surprising
**Why it works:** Builds mental model, prevents solving wrong problem

---

### 8. Mid-Stream Correction

**Description:** Redirect quickly when approach isn't working, without dwelling on failed attempt.

**How it shows up:**
- "i don't think it's working yet"
- "let's try a different approach"
- "actually, skip that and do X instead"

**Key:** Don't explain why the old approach failed unless asked. Just redirect.

---

## Velocity Anti-Patterns

### Ambiguous "Do It Again"

**Trap:** Repeating "there's more changes, do the same" without specifying which files.

**Signal:** Executor commits files you didn't intend, or misses expected files.

**Fix:** At minimum, review staged files before confirming. Or specify: "commit changes in [files] with message about [feature]"

---

### Skipped Verification Loop

**Trap:** Multiple commits pushed without checking running application.

**Signal:** Bugs discovered after they've been pushed.

**Fix:** After every 2-3 commits, verify actual behavior (run app, check output)

---

### Context Loss Through Fragmentation

**Trap:** Starting in one directory, asking about another, creating confusion.

**Signal:** Executor asks "which directory do you mean?"

**Fix:** Start each task block with context: "In [repo], I want to..."

---

### Over-Delegation of Design

**Trap:** Saying "go do it" for features without specifying acceptance criteria.

**Signal:** Result works but doesn't match your vision.

**Fix:** For non-trivial features: "Success looks like [behavior]. Key constraints: [list]."

---

## Mode Selection Guide

| Situation | Mode | Reasoning |
|-----------|------|-----------|
| Shipping known feature | Velocity | Territory is familiar, iterate fast |
| Debugging mystery bug | Deep | Need hypothesis-verify loops |
| Refactoring | Velocity | Patterns are clear, commit often |
| New architecture | Deep | High stakes, need understanding |
| UI tweaks | Velocity | Easy to see results, iterate |
| Data migration | Deep | High stakes, hard to rollback |
| Adding tests | Velocity | Clear patterns, incremental |
| Security fix | Deep | Must verify thoroughly |

---

## Quick Reference

**Velocity Mode Checklist:**
```
[ ] Goal is clear in one sentence
[ ] Territory is familiar
[ ] Stakes per action are low
[ ] Rollback is easy
[ ] Iteration beats analysis
```

**Velocity Commands:**
- "commit staged with proper message"
- "push it"
- "do the same for new changes"
- "bring that to [env] so I can validate"
- "do you see any gaps?"

**Switch to Deep Mode when:**
- You say "I don't understand why..."
- Same fix didn't work twice
- Stakes suddenly feel higher
- You're debugging not shipping

---

## Recipes

### Batch Commit Recipe

**Trigger:** User explicitly requests batch commit (e.g., "commit these changes", "batch commit", "commit all of this")

**Prerequisites (ALL must be true):**
- [ ] Build passes
- [ ] Tests pass
- [ ] User has verified behavior works
- [ ] User explicitly requested commit

**Process:**
1. Show staged files: `git status`
2. Ask user to confirm files are correct
3. Generate appropriate commit message
4. Execute commit only after user approval
5. Show result

**Safety Gates:**
- Never auto-commit without explicit user request
- Always show what will be committed before committing
- If unsure about scope, ask: "Should I commit [specific files] or everything staged?"

**Anti-pattern:** Committing after each small change without verification. This creates noisy history and potentially broken commits.
