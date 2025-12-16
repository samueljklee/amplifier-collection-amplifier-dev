# Workflow Patterns

> Task decomposition and progress tracking patterns for development workflow

---

## Overview

These patterns complement agent orchestration (from developer-expertise) with direct task management. Use these when working directly on tasks, not delegating to agents.

**When to use these patterns:**
- Breaking down complex work into trackable steps
- Managing progress visibility for the user
- Ensuring nothing falls through the cracks
- Maintaining momentum across multi-step tasks

**When to use agent delegation instead:**
- Architecture decisions → zen-architect
- Debugging → bug-hunter
- Implementation from specs → modular-builder
- Post-task cleanup → post-task-cleanup

---

## Task Decomposition Patterns

### 1. Explicit Todo Breakdown

**Description:** Convert complex requests into explicit, trackable todo items before starting work.

**How it works:**
- Receive task → Analyze scope → Create todo list → Work through items
- Each todo should be completable in one focused effort
- Todos should be ordered by dependency (what must come first)

**Example:**
```
User: "Add authentication to the API"

Todos created:
1. Research existing auth patterns in codebase
2. Design auth middleware approach
3. Implement auth middleware
4. Add auth to protected routes
5. Write tests for auth flow
6. Update API documentation
```

**When to use:** Any task with 3+ distinct steps
**When to skip:** Single-action tasks ("fix this typo")

---

### 2. Real-Time Progress Updates

**Description:** Mark todos complete immediately after finishing, not in batches.

**How it works:**
- Complete task → Mark done → Move to next
- Never batch completions ("I did 1-5, marking all done")
- User sees progress in real-time

**Why it matters:**
- User has visibility into actual progress
- Prevents "I thought you were done" confusion
- Creates natural checkpoints for course correction

**Anti-pattern:** Completing 5 tasks then marking all done at once. User can't see progress and may interrupt unnecessarily.

---

### 3. Scope Expansion Capture

**Description:** When new work is discovered during a task, capture it as new todos rather than silently expanding scope.

**How it works:**
- Discover new requirement → Add as new todo → Continue current task
- Don't silently expand what you're doing
- Let user see scope growing

**Example:**
```
Working on: "Add auth middleware"
Discover: "Need to add rate limiting too"

Action: Add new todo "Add rate limiting to auth"
Continue: Current auth middleware task
```

**Why it matters:**
- Prevents scope creep without visibility
- User can prioritize or defer new work
- Original task stays focused

---

### 4. Dependency-Ordered Execution

**Description:** Order todos by what must complete before other things can start.

**How it works:**
- Identify dependencies between tasks
- Order so blockers come first
- Note when tasks can be parallelized

**Example:**
```
Correct order:
1. Design schema (blocks 2, 3)
2. Implement models (blocks 4)
3. Implement API endpoints (blocks 4)
4. Write integration tests (needs 2, 3)

Parallelizable: 2 and 3 can run together after 1
```

**When to use:** Multi-step implementation tasks
**When to skip:** Independent tasks with no dependencies

---

### 5. Checkpoint Summarization

**Description:** After completing a logical group of todos, summarize progress before continuing.

**How it works:**
- Complete related todos → Summarize what's done → Confirm next steps
- Natural pause for user to redirect if needed
- Prevents long silent execution

**Example:**
```
Completed: Schema design, model implementation
Summary: "Database layer is complete. Models created for User, Session, Token. Ready to implement API endpoints."
Next: "Shall I proceed with the API endpoints?"
```

**When to use:** After completing a phase of work (e.g., "backend done, moving to frontend")
**When to skip:** When tasks are small and continuous

---

## Progress Visibility Patterns

### 6. Work-in-Progress Indicators

**Description:** Keep exactly one todo "in progress" at a time.

**How it works:**
- Start task → Mark "in progress"
- Complete → Mark "done" → Start next → Mark "in progress"
- Never have 0 or 2+ items in progress

**Why it matters:**
- User knows exactly what you're working on
- Clear signal of current focus
- Prevents parallel work without visibility

---

### 7. Blocker Escalation

**Description:** When blocked, immediately surface the blocker rather than silently struggling.

**How it works:**
- Hit blocker → Stop → Report blocker → Ask for guidance
- Don't spend cycles trying to work around unclear requirements
- User may have context that unblocks quickly

**Example:**
```
Blocker: "The API spec mentions 'legacy auth mode' but I can't find documentation. Should I: (a) skip it, (b) implement a placeholder, or (c) do you have more context?"
```

**When to use:** When uncertain about requirements or approach
**When to skip:** Technical blockers you can solve yourself

---

## Integration with Agent Delegation

When working with developer-expertise agents:

| Situation | Direct Work (these patterns) | Agent Delegation |
|-----------|------------------------------|------------------|
| Breaking down tasks | Use Explicit Todo Breakdown | - |
| Architecture decisions | - | zen-architect |
| Implementation | Use Progress Visibility | modular-builder |
| Debugging | Use Blocker Escalation | bug-hunter |
| Post-completion | - | post-task-cleanup |

**Handoff pattern:**
1. Break down task (these patterns)
2. Delegate architecture to zen-architect
3. Track implementation progress (these patterns)
4. Delegate debugging if stuck (bug-hunter)
5. Delegate cleanup (post-task-cleanup)

---

## Quick Reference

**Before starting complex work:**
```
[ ] Created explicit todo list
[ ] Ordered by dependencies
[ ] Each todo is one focused effort
```

**During work:**
```
[ ] Exactly one todo in progress
[ ] Completing items as I go (not batching)
[ ] Capturing new scope as new todos
```

**At checkpoints:**
```
[ ] Summarized progress
[ ] Confirmed next steps with user
[ ] Escalated any blockers
```
