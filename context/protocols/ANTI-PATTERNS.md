# Thinking Anti-Patterns

> Common thinking traps to recognize and avoid

---

## Overview

Anti-patterns are recurring mistakes that slow you down or lead to poor outcomes. Each entry includes:

- **How It Shows Up** - What the behavior looks like
- **"You're in this trap when..."** - Detection signal
- **Instead, Try...** - Better replacement move

---

## 1. Premature Confidence

**How It Shows Up:**
Declaring a fix works before validating, then user reports it's still broken.

**You're in this trap when...**
You say "this should work" or "that fixed it" without running tests or checking for secondary causes.

**Instead, Try:**
- Run validation immediately after implementing
- Ask yourself: "What could make this fail even after my fix?"
- Search for the same pattern elsewhere before declaring victory

---

## 2. Single-Source Verification

**How It Shows Up:**
Only checking one evidence source before proceeding with confidence.

**You're in this trap when...**
You read one file or run one test and assume everything is fine.

**Instead, Try:**
Use multi-channel verification:
- Code inspection (read the changed code)
- Tests (run them)
- Grep for old patterns (ensure complete removal)
- Check related areas (look for collateral damage)

---

## 3. Dependency Paralysis

**How It Shows Up:**
Getting blocked because "proper" tests require unavailable dependencies.

**You're in this trap when...**
You're not making progress because you can't run the full test suite, and you've stopped trying.

**Instead, Try:**
Create standalone validation that tests the specific change without full dependencies:
- Regex-based source file validation
- Minimal scripts that don't import the blocked module
- Unit tests with mocks for unavailable dependencies

---

## 4. Ignoring the Tests

**How It Shows Up:**
Fixing code without checking if tests assert the old behavior.

**You're in this trap when...**
Your fix is correct but tests fail because they expected the bug.

**Instead, Try:**
Always ask: "Do any tests assume the old behavior?"
- Grep for the old pattern in test files
- Read test assertions related to your change
- Update tests that were checking for the wrong thing

---

## 5. Inventing When Copying Works

**How It Shows Up:**
Creating novel implementations when sibling projects have working solutions.

**You're in this trap when...**
You're designing from scratch while similar code exists in nearby projects or files.

**Instead, Try:**
1. First search for reference implementations
2. Check sibling projects for how they solve similar problems
3. Copy patterns before inventing new ones
4. Only deviate when you have specific evidence the existing pattern is wrong

---

## 6. Confidence-Without-Evidence

**How It Shows Up:**
Saying "I'm confident" without citing specific evidence.

**You're in this trap when...**
You're asked for confidence and respond with assertions rather than citations.

**Instead, Try:**
Cite specifics:
- File names and line numbers
- Documentation links
- Multiple supporting examples
- Acknowledge remaining uncertainties

**Good:** "I'm confident because: (1) the OpenAI docs show X, (2) your test file at line 29 uses Y, (3) the error explicitly says Z."

**Bad:** "Yes, I'm confident this will work."

---

## 7. All-or-Nothing Validation

**How It Shows Up:**
Either running full test suites or no tests at all.

**You're in this trap when...**
You skip validation entirely because the "real" tests can't run right now.

**Instead, Try:**
Create minimal validation for the specific change:
- Something beats nothing
- A simple script that checks the specific behavior
- Manual verification with documented steps
- Standalone tests that don't require full environment

---

## 8. Batching Progress Updates

**How It Shows Up:**
Completing multiple tasks then marking them all done at once.

**You're in this trap when...**
You finish 5 tasks, then mark todos 1-5 complete in one batch.

**Instead, Try:**
- Mark each todo complete immediately after finishing it
- User sees real-time progress
- Creates natural checkpoints for course correction

**Why it matters:** User can't see progress and may interrupt unnecessarily, or assume you're stuck when you're actually making progress.

---

## 9. Permission Grinding

**How It Shows Up:**
Repeatedly trying the exact same approach after permission denied.

**You're in this trap when...**
Permission denied → try same thing → denied again → try same thing...

**Instead, Try:**
After 2-3 denials of the same approach:
1. Stop and reassess
2. Try an alternative approach
3. Ask user for guidance: "I'm being blocked from X. Should I try Y instead, or do you want to adjust permissions?"

**Threshold:** 2-3 attempts of the same approach before pivoting. Don't immediately pivot on first denial (might be transient), but don't grind indefinitely.

---

## Anti-Pattern Summary

| Anti-Pattern | Detection Signal | Replacement |
|--------------|------------------|-------------|
| Premature Confidence | "This should work" without testing | Validate immediately |
| Single-Source Verification | Only checked one thing | Multi-channel verification |
| Dependency Paralysis | Can't run full tests, stopped trying | Standalone validation |
| Ignoring the Tests | Fix correct, tests fail | Check if tests assert old behavior |
| Inventing When Copying Works | Designing from scratch | Search for reference implementations |
| Confidence-Without-Evidence | Assertions without citations | Cite files, lines, docs |
| All-or-Nothing Validation | Skip validation entirely | Minimal targeted validation |
| Batching Progress Updates | Mark 5 todos done at once | Mark complete immediately |
| Permission Grinding | Same denial 3+ times | Pivot after 2-3 attempts |

---

## Self-Check Questions

Before proceeding with a change, ask yourself:

1. Have I validated this, or just assumed it works?
2. Did I check multiple sources, or just one?
3. Is there existing code I should copy instead of inventing?
4. Could any tests be asserting the old (wrong) behavior?
5. Am I citing evidence, or just making assertions?
6. If full tests can't run, have I created minimal validation?
7. Am I marking progress in real-time, or batching updates?
8. Have I tried this same blocked approach too many times?
