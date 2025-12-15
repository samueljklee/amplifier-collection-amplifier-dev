# Understanding Amplifier Architecture Through First Principles

## Purpose

This framework helps reason about Amplifier's architecture by applying its core philosophical principles to any architectural question or decision.

## The Kernel Philosophy Lens

When encountering architectural decisions, evaluate through these questions:

### Mechanism vs. Policy Question

**Ask:**
- Is this a mechanism (how things CAN work)?
- Or a policy (how things SHOULD work)?

**Principle:** Mechanisms belong in kernel, policies belong in applications/modules.

**Examples:**
- ✅ Mechanism: "Hooks can observe events"
- ❌ Policy: "Hooks should log to console"
- ✅ Mechanism: "Modules can be loaded from sources"
- ❌ Policy: "Modules should come from git"

**Application:** If you're adding a feature, ask: "Is this a capability (mechanism) or a decision (policy)?" Capabilities go in kernel, decisions go in modules.

### Boundary Question

**Ask:**
- Where are the system boundaries?
- What data crosses each boundary?
- Is the boundary contract stable and documented?

**Principle:** Explicit boundaries with stable contracts enable independent evolution.

**Examples of boundaries:**
- Orchestrator → Coordinator (emits events)
- Coordinator → Hooks (delivers events)
- Hooks → DisplaySystem (calls methods)
- DisplaySystem → Application (emits to SSE/UI)

**Application:** If behavior is unexpected, identify which boundary it crosses. That's where to look for contract violations.

### Extension Question

**Ask:**
- Can this behavior be implemented as a module/hook?
- Or does it require kernel changes?

**Principle:** Default to "If it can be external, keep it external."

**Decision tree:**
```
Need new behavior?
├─ Can it be a new module? → Create module
├─ Can it be a hook? → Create hook
├─ Can it be a profile option? → Add to profile
└─ Requires kernel change → Strong justification needed
```

**Application:** Before proposing a kernel change, prove you can't solve it with modules.

## The Source of Truth Principle

When documentation and reality conflict, use this hierarchy:

1. **Code is truth** - What actually happens
2. **Tests are documentation** - What's expected to happen
3. **Contracts are specifications** - What must happen
4. **Written docs are intent** - What was meant to happen

**How to use:**
```
Found a discrepancy?
├─ Check the code: What does it actually do?
├─ Check the tests: What do they expect?
├─ Check the contract: What's required?
└─ Update the docs: Align with reality
```

**Anti-pattern:** Arguing about what docs say when code does something different.
**Correct approach:** Check code, understand why, update docs to match.

## The Composition Mindset

Amplifier favors composition over configuration.

**Configuration thinking (avoid):**
```yaml
behavior: streaming  # Flag to change behavior
max_retries: 3       # Number to tune behavior
```

**Composition thinking (prefer):**
```yaml
orchestrator: loop-streaming    # Swap component
hooks: [my-retry-hook]          # Add component
```

**How to apply:**
- Don't look for a flag to change behavior
- Look for a module to swap out
- Think: "What component provides this behavior?"
- Not: "What configuration enables this behavior?"

**Examples:**
- ❌ "How do I enable streaming?" → Looking for a flag
- ✅ "Which orchestrator provides streaming?" → Looking for a component
- ❌ "How do I configure logging?" → Looking for settings
- ✅ "Which hook provides logging?" → Looking for a component

## The Observability Question

When behavior is unclear, ask: "Can I observe it without modifying it?"

**Amplifier provides observability mechanisms:**
- Events are emitted at boundaries
- Hooks can observe without changing behavior
- Coordinator mediates all interactions

**How to use:**
```python
# Don't modify code to debug
# Add observation hooks instead:

async def debug_hook(event: str, data: dict):
    print(f"Event: {event}, Data: {data}")
    return HookResult(action="continue")

coordinator.hooks.register("*", debug_hook)  # Observe ALL events
```

**Why:** Observability without modification means you're debugging the real system, not a modified one.

## Key Architectural Distinctions

### Profile vs. Mount Plan vs. Session

**They're stages in a pipeline, not synonyms:**

```
Profile (human-readable)
  ↓ compile
Mount Plan (machine-readable)
  ↓ instantiate
Session (runtime)
```

- **Profile:** YAML + Markdown configuration
- **Mount Plan:** Dict with module specs
- **Session:** Running instance with loaded modules

**When to use each:**
- Debugging profile syntax → Check Profile parsing
- Debugging module loading → Check Mount Plan compilation
- Debugging runtime behavior → Check Session execution

### DisplaySystem vs. Hooks vs. UI

**Three separate layers:**

```
UI Layer (Application)
  ↑ receives events via
DisplaySystem (Protocol)
  ↑ called by
Hooks (Observers)
  ↑ receive events from
Orchestrator (Kernel)
```

- **DisplaySystem:** Protocol for kernel → app communication (injected FROM app)
- **Hooks:** Observation points within kernel (registered BY app)
- **UI:** Application-layer rendering (lives IN app)

**Common mistake:** Expecting DisplaySystem to be part of kernel. It's injected by the application.

### Source vs. Module vs. Entry Point

**Three concepts in module resolution:**

```
Source (git URL)
  ↓ resolver fetches
Module (Python package)
  ↓ discovered via
Entry Point (Python mechanism)
```

- **Source:** Where to GET a module (git, file, package)
- **Module:** The actual CODE satisfying a contract
- **Entry Point:** Python's DISCOVERY mechanism

**Resolution flow:**
1. Profile specifies source: `git+https://...`
2. Resolver fetches and caches module
3. Entry point allows Python to discover it
4. Loader imports and calls mount()

## Thinking in Layers

```
Application Layer (Your Code)
  ↓ injects UX systems (DisplaySystem, ApprovalSystem)
Kernel Layer (amplifier-core)
  ↓ emits events, coordinates modules
Module Layer (orchestrators, tools, hooks, etc.)
  ↓ uses capabilities from
Provider Layer (LLM APIs, external services)
```

**Problem-solving by layer:**
- Problem in LLM output? → Provider layer
- Problem with tool execution? → Module layer
- Problem with event delivery? → Kernel layer
- Problem with UI rendering? → Application layer

**Don't skip layers:** If the UI doesn't show something, don't immediately blame the LLM. Trace through each layer.

## Reasoning About Streaming

A common question: "Why isn't streaming working?"

**Apply the frameworks:**

1. **Mechanism vs. Policy:** Streaming is a mechanism (events). How you display it is policy (UI).
2. **Boundary question:** Events cross orchestrator → hooks → DisplaySystem → UI boundaries.
3. **Observability:** Can you observe events being emitted? Hook into them to verify.
4. **Composition:** Is the right orchestrator (loop-streaming) composed in?
5. **Layer thinking:** Which layer is failing? Events not emitted? Hooks not registered? DisplaySystem not called?

**Systematic diagnosis:**
```
Is orchestrator emitting events?
  ↓ No → Wrong orchestrator or config
  ↓ Yes
Are hooks receiving events?
  ↓ No → Hooks not registered or wrong event names
  ↓ Yes
Is DisplaySystem being called?
  ↓ No → Hooks not calling DisplaySystem (need bridge hook)
  ↓ Yes
Is UI receiving DisplaySystem output?
  ↓ No → DisplaySystem not emitting to UI correctly
  ↓ Yes
Is UI rendering correctly?
  ↓ No → UI layer bug
```

## When Architecture Seems Wrong

If Amplifier's architecture seems wrong or backwards:

1. **Re-read KERNEL_PHILOSOPHY.md** - Understand the "why"
2. **Check if you're thinking in configuration** - Should be thinking in composition
3. **Verify you understand the boundaries** - What crosses where?
4. **Look for working examples** - How do they do it?
5. **Question your assumptions** - Is documentation current?

**Remember:** The architecture optimizes for:
- Kernel stability (center stays still)
- Module replaceability (edges move fast)
- Clear boundaries (explicit contracts)
- Observability (events everywhere)

If your solution requires violating these, rethink the solution.
