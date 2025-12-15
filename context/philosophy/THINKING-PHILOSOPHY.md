# Thinking Philosophy

> Evidence-driven, iterative problem solving through systematic reasoning

---

## Core Mindset

### 1. Evidence Over Intuition

When asked "are you confident?", treat it as a prompt to gather more evidence, not to defend your position.

**Do:**
- Cite specific files and line numbers
- Reference documentation links
- Show multiple supporting evidence sources
- Acknowledge uncertainty when it exists

**Don't:**
- Say "I'm confident" without citations
- Defend positions without evidence
- Assume correctness without verification

### 2. Reference Before Inventing

Before implementing a new feature or pattern, first examine how similar things are done elsewhere.

**The heuristic:**
```
Similar code exists nearby? → Copy the pattern
No similar code? → Research first, then implement
Multiple patterns exist? → Create comparison table, follow majority
```

**Why this matters:**
- Ensures ecosystem consistency
- Reduces decision fatigue
- Leverages proven solutions
- Makes code review easier

### 3. Validation Has Layers

A single passing test isn't enough. Validate through multiple channels:

| Channel | What It Catches |
|---------|-----------------|
| **Run tests** | Regressions, broken contracts |
| **Grep for old patterns** | Incomplete refactors |
| **Check related areas** | Collateral damage |
| **Version verification** | "Works on my machine" issues |

### 4. When Blocked, Widen

If the obvious fix didn't work, the problem is often elsewhere:
- Tests enforcing old (wrong) behavior
- Version mismatches
- Duplicate code you didn't find
- Configuration overrides

---

## The Confidence Spectrum

Not all changes require the same rigor:

| Stakes | Verification Level |
|--------|-------------------|
| **Low** (typo fix, comment) | Single verification |
| **Medium** (bug fix, small feature) | Multi-channel verification |
| **High** (architecture, breaking change) | Evidence gathering + human confirmation |

### Confidence Gating

When stakes are high OR when explicitly challenged:

1. **Pause** - Don't proceed immediately
2. **Gather** - Search for counter-evidence
3. **Cite** - List specific evidence sources
4. **Acknowledge** - Note remaining uncertainties
5. **Proceed** - Only with sufficient evidence

---

## Decision Heuristics

### When multiple approaches exist:
1. Check what sibling projects do
2. Create comparison table if variance is high
3. Prefer the majority pattern unless you have specific evidence it's wrong

### When deciding keep vs. delete:
1. Assess standalone value
2. Consider maintenance burden
3. Check if anyone depends on it
4. Prefer deletion when in doubt (less code = fewer bugs)

### When adding dependencies:
1. Check if sibling projects include it
2. Assess if it's runtime vs. dev dependency
3. Consider circular dependency risks
4. Don't add dependencies that aren't in similar projects without strong justification

---

## The Learning Loop

After each session:

1. **What patterns worked?** → Document in PATTERNS.md
2. **What traps did I fall into?** → Document in ANTI-PATTERNS.md
3. **What unblocked me?** → Add to UNBLOCKING-PLAYBOOK.md

This collection should grow with your experience.
