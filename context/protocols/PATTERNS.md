# Thinking Patterns

> Proven reasoning patterns extracted from real development sessions

---

## Overview

These patterns are organized into 5 categories:
- **Problem Framing** - How to understand what you're solving
- **Decomposition & Planning** - How to break down work
- **Unblocking** - How to get unstuck
- **Decision Making** - How to choose between options
- **Validation** - How to verify your work

Each pattern includes:
- **Description** - What it is
- **Example** - How it shows up in practice
- **Usefulness** - When it helps vs. when it's costly
- **Domain** - Whether it's general or context-specific

---

## A. Problem Framing Patterns

### 1. Error-First Tracing
**Description:** Start from the exact error message and trace backward to the source code.

**Example:** User pastes `unexpected keyword argument 'system'`; immediately grep for `params["system"]` to find the offending code.

**Usefulness:**
- **Helpful** when error messages are informative
- **Less useful** for vague errors like "something went wrong"

**Domain:** Agnostic

---

### 2. Confidence Gating
**Description:** When challenged on confidence ("are you sure?"), pause to gather additional evidence before proceeding.

**Example:** Human asks "are you confident?"; respond with 4 specific evidence sources (file names, line numbers, docs) before confirming.

**Usefulness:**
- **Helpful** - Catches premature conclusions
- **Costly** if overused on low-stakes changes

**Domain:** Agnostic

---

### 3. Explicit Problem Restatement
**Description:** After diagnosing or fixing, summarize what was wrong and what changed.

**Example:** "Root Cause: X was passing Y to Z, which doesn't support it. Solution: Changed to pass Y via W instead."

**Usefulness:**
- **Helpful** - Creates shared understanding
- **Useful** for handoffs and future debugging

**Domain:** Agnostic

---

## B. Decomposition & Planning Patterns

### 4. Reference Implementation Mining
**Description:** Before building, examine 2+ existing implementations of similar features.

**Example:** Before implementing debug modes, check Anthropic, Ollama, Azure, vLLM providers to see how they do it.

**Usefulness:**
- **Helpful** - Ensures consistency, reduces reinvention
- **Costly** if reference implementations are poor or unavailable

**Domain:** Agnostic

---

### 5. Sequential Capability Stacking
**Description:** Add one capability at a time: config → helper → core logic → docs → tests.

**Example:** Debug mode implemented in exactly this order across multiple turns, validating each step before moving on.

**Usefulness:**
- **Helpful** - Each step validates before the next
- **Can feel slow** but reduces rework

**Domain:** Agnostic

---

### 6. Table-Based Comparison
**Description:** When variance exists across implementations, create a comparison table to visualize differences.

**Example:**
| Provider | Has Feature? | Implementation Type |
|----------|--------------|---------------------|
| Anthropic | Yes | Instance method |
| Ollama | Yes | Module function |
| Azure | No | N/A |

**Usefulness:**
- **Helpful** - Makes tradeoffs visible
- **Overkill** for binary choices

**Domain:** Agnostic

---

## C. Unblocking Patterns

### 7. Widen-the-Net Search
**Description:** If fix didn't work, search for the same pattern in tests, configs, sibling files.

**Example:** Error persists after fix → grep finds tests were asserting old (wrong) behavior.

**Usefulness:**
- **Helpful** - Often catches secondary causes

**Domain:** Agnostic

---

### 8. Standalone Validation Bypass
**Description:** When dependencies block testing, create a minimal test that avoids them entirely.

**Example:** `amplifier_core` unavailable → create regex-based source validation test that reads the file directly.

**Usefulness:**
- **Helpful but costly** - Unblocks progress but creates technical debt
- **Use sparingly** - These tests may not run in CI

**Domain:** Specific (testing)

---

### 9. Version Pinpointing
**Description:** When behavior seems version-dependent, explicitly verify the installed version.

**Example:** Show 3 methods to check SDK version: lockfile, cache inspection, dry-run sync.

**Usefulness:**
- **Helpful** - Catches "works on my machine" issues

**Domain:** Specific (Python/packaging)

---

## D. Decision Making Patterns

### 10. Follow-the-Herd Default
**Description:** When multiple valid approaches exist, prefer what sibling projects do.

**Example:** "Should we add amplifier_core as dependency?" → Check other providers, find none do, recommend against.

**Usefulness:**
- **Helpful** - Reduces decision fatigue, ensures ecosystem consistency
- **Risk** - Can perpetuate bad patterns if the herd is wrong

**Domain:** Agnostic

---

### 11. Keep-vs-Delete Pragmatism
**Description:** Decide based on standalone value vs. maintenance burden.

**Example:** Keep comprehensive tests despite official providers using simpler pattern, because they provide standalone value.

**Usefulness:**
- **Helpful** - Balances thoroughness with practicality

**Domain:** Agnostic

---

### 12. Interface Simplicity Principle
**Description:** Keep module interfaces minimal. Callers provide context, modules provide capability.

**Example:** Instead of a function that guesses what the user wants:
```python
# Complex: Module tries to be smart
def process(input, maybe_format=None, auto_detect=True, fallback=True):
    if auto_detect:
        format = guess_format(input)
    ...

# Simple: Caller provides context
def process(input, format):
    ...
```

**Usefulness:**
- **Helpful** - Reduces ambiguity, makes behavior predictable
- **Aligns with** foundation's "minimal abstractions" principle

**Domain:** Agnostic

---

## E. Validation Patterns

### 13. Multi-Channel Verification
**Description:** Validate through code inspection AND tests AND documentation.

**Example:** Fix validated via: (1) grep for old pattern, (2) standalone test, (3) reinstall check.

**Usefulness:**
- **Helpful** - Catches issues that slip through single-channel validation
- **Costly** for trivial changes

**Domain:** Agnostic

---

### 14. Fix-the-Tests-Too
**Description:** When fixing code, update tests that were asserting old (wrong) behavior.

**Example:** Tests expected `params["system"]` to exist; updated them to check for messages array format instead.

**Usefulness:**
- **Helpful** - Prevents false failures
- **Easy to forget** - Always check for this

**Domain:** Specific (testing)

---

### 15. Grep-for-Ghosts
**Description:** After removing a pattern, grep to ensure no instances remain.

**Example:** After fix, grep for `params["system"]` to confirm complete removal across codebase.

**Usefulness:**
- **Helpful** - Catches incomplete refactors

**Domain:** Agnostic

---

## Pattern Summary Table

| # | Pattern | Helpful? | Domain |
|---|---------|----------|--------|
| 1 | Error-First Tracing | Yes | Agnostic |
| 2 | Confidence Gating | Yes | Agnostic |
| 3 | Explicit Problem Restatement | Yes | Agnostic |
| 4 | Reference Implementation Mining | Yes | Agnostic |
| 5 | Sequential Capability Stacking | Yes | Agnostic |
| 6 | Table-Based Comparison | Yes | Agnostic |
| 7 | Widen-the-Net Search | Yes | Agnostic |
| 8 | Standalone Validation Bypass | Yes (costly) | Specific |
| 9 | Version Pinpointing | Yes | Specific |
| 10 | Follow-the-Herd Default | Yes | Agnostic |
| 11 | Keep-vs-Delete Pragmatism | Yes | Agnostic |
| 12 | Interface Simplicity Principle | Yes | Agnostic |
| 13 | Multi-Channel Verification | Yes | Agnostic |
| 14 | Fix-the-Tests-Too | Yes | Specific |
| 15 | Grep-for-Ghosts | Yes | Agnostic |
