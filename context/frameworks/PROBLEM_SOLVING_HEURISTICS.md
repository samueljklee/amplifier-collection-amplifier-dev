# Problem-Solving Heuristics for Amplifier Development

## Purpose

These heuristics are mental shortcuts for solving common problems in Amplifier development. They're patterns that consistently lead to solutions.

## When Integration Fails

### Heuristic: "Work from Known-Good Outward"

**Pattern:** Find a working reference, then diff against it.

**Steps:**
1. Find a working example (CLI, another project, test)
2. Compare: What does it do that you don't?
3. Identify: What assumptions does it make?
4. Replicate: What order does it do things in?

**Example:**
```
Your integration fails, but amplifier-app-cli works.
→ How does CLI create sessions?
→ Does it mount a resolver? (Yes!)
→ Do you mount a resolver? (No!)
→ Solution found.
```

**Why it works:** Working code reveals implicit requirements that docs might miss.

### Heuristic: "Simplify to Minimal Reproduction"

**Pattern:** Strip complexity until it works, then add back piece by piece.

**Steps:**
1. Identify the smallest thing that should work
2. Remove everything else
3. Verify the minimal case works
4. Add complexity back one piece at a time
5. Identify what breaks it

**Example:**
```
Profile with 10 modules doesn't load.
→ Try with just orchestrator + context (minimal session)
→ Works? Add providers
→ Works? Add tools one by one
→ Breaks? Found the problematic module
```

**Why it works:** Removes confounding variables. You know exactly what caused the failure.

### Heuristic: "Check the Prerequisites"

**Pattern:** Verify setup before debugging behavior.

**Checklist:**
- [ ] Dependencies installed?
- [ ] Paths correct?
- [ ] Entry points registered?
- [ ] Environment configured?
- [ ] Modules discoverable?

**Example:**
```
"Module not found" error
→ Before debugging module loading logic...
→ Can you import the module directly?
→ Is it in entry points?
→ Is the cache populated?
```

**Why it works:** 80% of "bugs" are actually "missing setup." Check the obvious first.

## When Behavior is Unexpected

### Heuristic: "Events Don't Lie"

**Pattern:** If you think something should happen but doesn't, observe the events.

**Steps:**
1. Hook the relevant events
2. Log what actually fires
3. Compare expectation vs. reality
4. Adjust your mental model

**Example:**
```python
# You expect content_block:delta events
# Add observer:
async def observe(event, data):
    print(f"Event: {event}")
    return HookResult(action="continue")

coordinator.hooks.register("*", observe)

# Run and see what ACTUALLY fires
# Adjust expectations based on reality
```

**Why it works:** Events are the ground truth. They show what the system actually does.

### Heuristic: "Boundary Testing"

**Pattern:** Don't test the whole system. Test each boundary independently.

**Steps:**
1. Identify the boundaries data crosses
2. Test input arrives correctly at boundary
3. Test output emerges correctly from boundary
4. Only then test the full path

**Example:**
```python
# Don't test: User input → LLM → Tool → Response
# Instead, test each boundary:

# Boundary 1: User input → Session
assert session.can_accept_input()

# Boundary 2: Session → Orchestrator  
assert orchestrator.received_prompt()

# Boundary 3: Orchestrator → Tool
assert tool.was_called()

# Then test full path
```

**Why it works:** Isolates the failure point. You know which boundary failed.

### Heuristic: "API Evolution Awareness"

**Pattern:** When "something that should work" doesn't, check if the API evolved.

**Steps:**
1. Check version of library you're using
2. Look for deprecation warnings
3. Compare with examples from same version
4. Check if methods/signatures changed

**Example:**
```python
# Old API: resolver.collections.values()
# New API: resolver.list_collections()

# If you're using old API with new version → fails silently
# Solution: Check CHANGELOG or commit history
```

**Why it works:** APIs evolve. Documentation can lag. Version mismatches are common.

## When Documentation is Unclear

### Heuristic: "Read the Contracts"

**Pattern:** Specifications > Explanations > Examples

**Priority order:**
1. **Contracts** - Define MUST behavior (authoritative)
2. **Tests** - Show expected behavior (executable)
3. **Docs** - Explain WHY (contextual)
4. **Examples** - Show HOW (illustrative)

**Example:**
```
Question: "How should my hook behave?"
→ Read HOOK_CONTRACT.md (must return HookResult)
→ Read hook tests (shows expected patterns)
→ Read hook docs (explains use cases)
→ Read hook examples (shows implementation)

Don't skip to examples first.
```

**Why it works:** Contracts define correctness. Everything else is guidance.

### Heuristic: "Follow the Types"

**Pattern:** Type signatures reveal intent.

**Questions to ask:**
- What does the function accept?
- What does it return?
- What does `None` mean vs. a value?
- What does `Optional` tell you?

**Example:**
```python
async def mount(coordinator, config: dict) -> Callable | None:
    #                                      ^^^^^^^^^^^^^^^^
    # Returns cleanup function OR None
    # None means no cleanup needed
    # Callable means cleanup required
```

**Why it works:** Types are executable documentation. They're always correct (or code won't run).

## When Stuck on Implementation

### Heuristic: "Ask 'What's the Simplest Thing?'"

**Pattern:** Resist clever solutions. Start with the obvious.

**Steps:**
1. State the goal clearly
2. Ignore performance, elegance, future-proofing
3. Write the simplest code that works
4. Refactor only if needed

**Example:**
```python
# Goal: Forward events to DisplaySystem
# Don't immediately build: Event router, transformer, queue, etc.

# Simplest thing:
async def forward(event, data):
    await display_system.show_message(data['text'])
    return HookResult(action="continue")

# Works? Ship it. Complexity can come later if needed.
```

**Why it works:** Simple solutions are easier to debug, test, and modify.

### Heuristic: "Spike Before Committing"

**Pattern:** Prototype quickly to test assumptions before building the real thing.

**Steps:**
1. Identify the risky assumption
2. Write throwaway code to test it
3. Verify it works (or doesn't)
4. Now write it properly

**Example:**
```python
# Assumption: "I can hook into content_block:delta"
# Spike:
coordinator.hooks.register("content_block:delta", lambda e,d: print(d))
# Run and see if it prints

# If it works → build the real hook
# If it doesn't → question the assumption
```

**Why it works:** Validates assumptions cheaply before investing in implementation.

### Heuristic: "Grep Before Implementing"

**Pattern:** See how existing code solves similar problems.

**Steps:**
1. Identify what you're trying to do
2. Grep for similar patterns in codebase
3. Understand the existing solution
4. Adapt it to your needs

**Example:**
```bash
# Need to implement a hook?
# Grep for existing hooks:
grep -r "async def mount" ~/.amplifier/module-cache/

# See how they register, what they return, etc.
# Copy the pattern, adapt the behavior
```

**Why it works:** Existing code shows working patterns. Don't reinvent.

## When Everything Seems Wrong

### Heuristic: "Take a Step Back"

**Pattern:** When deep in details, zoom out to the bigger picture.

**Questions to ask:**
- What am I actually trying to accomplish?
- Is this the right approach?
- Am I solving the right problem?
- What would I do if starting fresh?

**Example:**
```
Spent 2 hours debugging hook registration
→ Step back: "Why am I using hooks?"
→ Realize: "Maybe I should use DisplaySystem directly"
→ Simpler solution emerges
```

**Why it works:** Sometimes you're deep in the wrong solution. Zooming out reveals better paths.

### Heuristic: "Rubber Duck It"

**Pattern:** Explain the problem out loud (or in writing).

**Steps:**
1. State what you're trying to do
2. Explain what you've tried
3. Describe what's happening
4. Often, the solution becomes obvious while explaining

**Example:**
```
Writing: "I'm trying to get DisplaySystem to receive events..."
         "The orchestrator emits them..."
         "But nothing's calling DisplaySystem..."
         "Oh! I need a hook to bridge them!"
```

**Why it works:** Externalizing forces you to organize your thoughts. Gaps become visible.

## Meta-Heuristic: Pattern Recognition

After solving several problems, you'll notice patterns:

- "Module not found" → Check resolver mounting
- "Hangs on init" → Check module sources/network
- "Events not firing" → Check hook registration timing
- "Methods not called" → Check who's supposed to call them

**Build your own pattern library:**
```
When I see [symptom] → I check [cause] → Usually it's [issue]
```

These patterns become instinct with practice.

## When Heuristics Fail

If none of these heuristics lead to a solution:

1. **Question your assumptions** - Is your mental model wrong?
2. **Read the source code** - What does it actually do?
3. **Ask for help** - Fresh eyes see what you've become blind to
4. **Sleep on it** - Sometimes the solution comes when you stop thinking about it

Remember: These are heuristics, not algorithms. They guide you toward solutions but don't guarantee them. Use judgment.
