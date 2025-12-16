# Amplifier Design Principles for Development

## Purpose

These principles guide decision-making when building with or extending Amplifier. They distill the kernel philosophy into actionable guidance.

## Principle: Kernel Stability Over Feature Velocity

**Statement:** The kernel changes rarely. If your solution requires a kernel change, you're probably solving the wrong problem.

**Reasoning:** A stable kernel enables fast evolution at the edges. Kernel changes cascade to all modules and applications.

**Application:**
- ✅ Need new behavior? → Add/swap a module
- ✅ Need observability? → Add a hook
- ✅ Need customization? → Extend in application layer
- ❌ Kernel change? → Strong justification required

**Example:**
```
Problem: "Need to log all events"
❌ Bad: Modify kernel to add logging
✅ Good: Create hooks-logging module
```

**When to break:** Only when:
1. Multiple modules need the same kernel capability
2. It's a mechanism (not policy)
3. It maintains backward compatibility
4. Community consensus exists

## Principle: Least Surprise (Follow Patterns)

**Statement:** Amplifier follows standard patterns. If a solution feels "magical," it's probably wrong.

**Reasoning:** Predictability reduces cognitive load. Developers should recognize patterns from other systems.

**Patterns Amplifier follows:**
- Python entry points for module discovery
- Async/await for coordination
- Protocols for contracts
- Hooks for observation (middleware pattern)
- Dependency injection for UX systems

**Application:**
```
✅ Good: Module registered via entry point
❌ Bad: Module registered via magic import

✅ Good: Hook observes without modifying
❌ Bad: Hook changes behavior with side effects

✅ Good: DisplaySystem injected from application
❌ Bad: DisplaySystem auto-instantiated by kernel
```

**Anti-pattern:** "It just works" without understanding how.
**Better:** "I understand the mechanism that makes it work."

## Principle: Composition Over Configuration

**Statement:** Don't look for flags to change behavior. Look for modules to swap out.

**Reasoning:** Configuration scales poorly (exponential flag combinations). Composition scales linearly (add/remove modules).

**Configuration thinking (avoid):**
```yaml
features:
  streaming: true
  logging: verbose
  retries: 3
```

**Composition thinking (prefer):**
```yaml
orchestrator: loop-streaming  # Streaming behavior
hooks:
  - hooks-logging             # Logging behavior
  - hooks-retry               # Retry behavior
```

**Benefits of composition:**
- Modules are independently testable
- Behavior is explicit (not hidden in flag combinations)
- Modules can be version-controlled separately
- No combinatorial explosion of configurations

**When configuration is okay:** Module-internal settings that don't change behavior type.

## Principle: Explicit Over Implicit

**Statement:** Dependencies, capabilities, and contracts should be explicit. Magic is bugs waiting to happen.

**Reasoning:** Explicit systems are debuggable. Implicit systems fail mysteriously.

**Application:**

**Explicit dependencies:**
```python
# Good: Module declares what it needs
async def mount(coordinator, config):
    if not coordinator.has_provider():
        raise ConfigError("Requires at least one provider")
```

**Explicit contracts:**
```python
# Good: Function signature shows requirements
async def mount(coordinator: ModuleCoordinator, config: dict) -> Callable | None:
    ...
```

**Explicit sources:**
```yaml
# Good: Profile specifies where modules come from
tools:
  - module: tool-bash
    source: git+https://github.com/microsoft/amplifier-module-tool-bash@main
```

**Anti-patterns:**
- Globals that modules magically access
- Assumptions about environment without checking
- "It should just work" without explicit setup

## Principle: Mechanism Over Policy

**Statement:** The kernel provides capabilities. Applications make decisions.

**Reasoning:** Different applications have different needs. Kernel shouldn't impose policy.

**Mechanism (kernel provides):**
- Hooks CAN observe events
- Events ARE emitted at boundaries
- Modules CAN be loaded from sources

**Policy (application decides):**
- WHAT to log (hooks-logging decides)
- WHERE to log (application configures)
- WHEN to approve actions (ApprovalSystem decides)

**Application:**
```
Mechanism: Hooks receive all events
Policy: Which events to log is a hook decision

Mechanism: Orchestrator can emit thinking events  
Policy: Whether to display them is a UI decision

Mechanism: Modules can request approval
Policy: What to approve is an ApprovalSystem decision
```

**Litmus test:** If two reasonable teams would disagree on the behavior, it's policy (not mechanism).

## Principle: Observable By Default

**Statement:** Everything emits events. Everything can be hooked. Everything can be traced.

**Reasoning:** If you can't observe it, you can't debug it. Observability should be built in, not added later.

**Application:**
- All kernel boundaries emit events
- All events can be hooked
- All hooks can observe without modifying
- All module interactions are mediated (visible)

**Example:**
```python
# Kernel emits events at boundaries
coordinator.hooks.emit('module:loaded', {'module': name})
coordinator.hooks.emit('tool:pre', {'tool': tool_name})
coordinator.hooks.emit('session:end', {'session_id': id})

# Application can observe
coordinator.hooks.register('*', my_observer)
```

**Why this matters:** Debugging without observability is guessing.

## Principle: Text-First, Inspectable Surfaces

**Statement:** Favor human-readable, deterministic, and versionable representations.

**Reasoning:** Text can be diffed, version-controlled, read by humans, and processed by tools.

**Application:**
- Profiles are YAML + Markdown (text)
- Events are dicts with string keys (inspectable)
- Contracts are documented in text (not just code)
- Configuration is declarative (not imperative)

**Anti-patterns:**
- Binary configuration formats
- Opaque state that can't be inspected
- Magic that happens without artifacts

**Benefits:**
- Can git diff profiles to see changes
- Can inspect events in debugger
- Can understand system without running it

## Principle: Determinism Before Parallelism

**Statement:** Prefer simple, deterministic flows over clever concurrency.

**Reasoning:** Deterministic systems are predictable and debuggable. Parallelism adds complexity.

**Application:**
- Sequential by default
- Parallel when proven necessary
- Concurrent only when required for performance
- Always with clear ordering guarantees

**Example:**
```python
# Default: Sequential (deterministic)
for tool in tools:
    result = await tool.execute()

# Only if needed: Parallel (complex)
tasks = [tool.execute() for tool in tools]
results = await asyncio.gather(*tasks)
```

**When to parallelize:** After proving sequential is too slow, not before.

## Principle: Trust But Verify

**Statement:** Trust documentation and APIs, but verify they work as documented.

**Reasoning:** APIs evolve. Documentation lags. Reality is the ground truth.

**Application:**
```python
# Trust: API should work like docs say
# Verify: Does it actually?

# Example:
# Docs say: resolver.collections.values()
# Reality: resolver.list_collections()  # API evolved

# Always verify with actual code/tests
```

**Pattern:**
1. Read the documentation
2. Read the contract
3. Read the tests
4. Read the code if needed
5. Write your own test to verify

**Why:** Documentation describes intent. Code describes reality.

## Principle: Simplicity Is Sophistication

**Statement:** The simplest solution that works is usually the right one.

**Reasoning:** Simple code is easier to understand, modify, test, and debug.

**Application:**
- Resist clever solutions
- Avoid premature optimization
- Question every abstraction
- Prefer clear over concise

**Example:**
```python
# Simple (preferred)
async def forward_event(event, data):
    await display.show_message(data['text'])
    return HookResult(action='continue')

# Complex (avoid unless proven necessary)
class EventRouter:
    def __init__(self):
        self.handlers = {}
        self.transformers = []
    # ... 50 more lines of abstraction
```

**When complexity is justified:**
- Proven performance requirement
- Multiple consumers need the abstraction
- Complexity is isolated and contained

## Principle: Boundaries Are Contracts

**Statement:** Every boundary has a stable contract. Violating it breaks compatibility.

**Reasoning:** Stable boundaries enable independent evolution of components.

**Application:**
- Module mount signature is stable: `async def mount(coordinator, config)`
- Hook return is stable: `HookResult`
- Event data schemas are documented
- Breaking changes are explicitly versioned

**When you cross a boundary:**
1. Read the contract
2. Satisfy the contract exactly
3. Don't assume extra capabilities
4. Don't rely on undocumented behavior

**Why boundaries matter:** They're where integration happens - and where it breaks.

## Applying These Principles

When making a decision:

1. **Is it a mechanism or policy?** → Policy goes in modules/apps
2. **Does it follow patterns?** → Use standard patterns, not magic
3. **Can I compose instead of configure?** → Prefer composition
4. **Is it explicit?** → Make dependencies and contracts visible
5. **Is it observable?** → Ensure I can debug it
6. **Is it deterministic?** → Prefer simple over concurrent
7. **Is it simple?** → Simplest working solution wins
8. **Does it respect boundaries?** → Honor contracts

These principles work together to create systems that are:
- Easy to understand
- Easy to extend
- Easy to debug
- Hard to break

When in doubt, choose the option that best upholds these principles.
