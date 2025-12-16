# Thinking Playbook

> Complete 20-point checklist for systematic problem-solving

---

## Starting a Task

### 1. Clarify the goal in one sentence
What does "done" look like? If it's a bug, what exact error or behavior needs to change?

### 2. Identify constraints
What can't change? What dependencies exist? What patterns must be followed?

### 3. Define success criteria
How will you know the task is complete? (Tests pass? Error disappears? User confirms?)

---

## Making Progress

### 4. Search before building
Grep for relevant patterns. Read related files. Check sibling projects for reference implementations.

### 5. Form an explicit hypothesis
State what you think is wrong or what approach you'll take. Make it falsifiable.

### 6. Check confidence when stakes are high
If the change is significant or you're uncertain, pause to gather evidence. Cite specifics, not assertions.

### 7. Prefer existing patterns
Copy from sibling implementations before inventing. Consistency > novelty.

### 8. Make the smallest change that could work
Avoid bundling fixes. One change, one validation.

### 9. Stack capabilities sequentially
Config → helper → core logic → docs → tests. Validate each before moving on.

---

## Recovering When Stuck

### 10. Re-read the error literally
The clue is often in the exact wording.

### 11. Widen the search
Check tests, configs, other files for the same pattern.

### 12. Ask "what else could cause this?"
Generate alternative hypotheses before committing to one.

### 13. Create standalone validation
If dependencies block, test the specific change in isolation.

### 14. Check versions
SDK or library version may not support the API you're using.

---

## Closing the Loop

### 15. Validate through multiple channels
Run tests AND grep for old patterns AND check related areas.

### 16. Fix the tests too
If tests assert old (wrong) behavior, update them.

### 17. Grep for ghosts
After removing a pattern, confirm no instances remain.

### 18. Summarize what changed
"Root cause: X. Solution: Y." Creates shared understanding.

### 19. Update documentation
If behavior changed, docs should reflect it.

### 20. Confirm with the human
Explicit closure prevents assumptions about completion.

---

## Quick Reference Card

```
START
├── Clarify goal in one sentence
├── Identify constraints
└── Define success criteria

PROGRESS
├── Search before building
├── Form explicit hypothesis
├── Check confidence if high-stakes
├── Prefer existing patterns
├── Smallest change that could work
└── Stack sequentially: config → helper → logic → docs → tests

STUCK?
├── Re-read error literally
├── Widen search (tests, configs, siblings)
├── Generate alternative hypotheses
├── Create standalone validation
└── Check version mismatches

CLOSE
├── Multi-channel validation
├── Fix tests if they assert old behavior
├── Grep for ghosts (old patterns remaining)
├── Summarize what changed
├── Update docs
└── Confirm with human
```

---

## Decision Heuristics

| Situation | Heuristic |
|-----------|-----------|
| Multiple approaches exist | Follow what sibling projects do |
| Keep vs. delete | Assess standalone value vs. maintenance burden |
| Adding dependency | Don't add if siblings don't have it |
| Confidence challenged | Pause, gather evidence, cite sources |
| Validation failed | Widen search before repeating fix |

---

## The Core Loop (Simplified)

```
Clarify → Search → Hypothesize → Implement → Validate → Document
                        ↑                         │
                        └──── if broken ──────────┘
```

If validation fails, widen the search and loop back.
