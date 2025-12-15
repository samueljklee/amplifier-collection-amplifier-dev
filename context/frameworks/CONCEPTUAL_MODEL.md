# Mental Model for Amplifier Architecture

## Purpose

This document provides a conceptual model for understanding Amplifier's architecture - the mental framework that makes the system's design intuitive rather than mysterious.

## Core Abstractions

### Session = Coordination Context

A session is NOT a conversation history or a stateful agent. It's a **coordination context** that assembles capabilities for a specific interaction scope.

**Think of it as:**
- An orchestra conductor (not the musicians)
- A factory assembly line (not the products)
- A project workspace (not the files)

**What it does:**
- Coordinates modules for a conversation
- Provides infrastructure (hooks, events, coordinator)
- Manages lifecycle (initialize, execute, cleanup)

**What it doesn't do:**
- Store conversation history (context manager does this)
- Execute LLM calls (orchestrator does this)
- Implement business logic (modules do this)

### Module = Pluggable Capability

A module is a self-contained unit that provides a specific capability by satisfying a contract.

**Module types and their purposes:**

| Type | Purpose | Example |
|------|---------|---------|
| Orchestrator | Conversation flow strategy | loop-streaming, loop-basic |
| Provider | LLM backend interface | provider-anthropic, provider-openai |
| Tool | Actions the system can take | tool-bash, tool-filesystem |
| Hook | Observability & control | hooks-logging, hooks-streaming-ui |
| Context | Conversation memory strategy | context-simple, context-persistent |

**Key insight:** Modules are **composed**, not configured. You don't enable streaming with a flag - you use a streaming orchestrator.

### Hook = Observation Point

Hooks are observers that see what happens without changing what happens.

**Mental model:** Middleware pattern
```
Event → Hook₁ → Hook₂ → Hook₃ → Continue
         ↓        ↓        ↓
      observe  observe  observe
```

**What hooks CAN do:**
- Observe events
- Log, trace, record
- Emit their own events
- Return HookResult to continue/stop

**What hooks CANNOT do:**
- Modify event data (read-only)
- Change system behavior
- Call LLMs or tools directly
- Block the event chain (except via HookResult)

**Think of hooks as:** Security cameras - they watch and record, but don't interfere.

### Coordinator = Module Assembly Point

The coordinator is where modules connect to the system.

**Mental model:** A junction box with labeled sockets
```
Coordinator
├─ "orchestrator" socket → (one orchestrator)
├─ "context" socket → (one context manager)
├─ "providers" socket → (multiple providers)
├─ "tools" socket → (multiple tools)
└─ "hooks" socket → (hook chain)
```

**What it provides:**
- Stable mount points (sockets) for modules
- Mediation between modules
- Hook chain management
- Enforcement of kernel invariants

**What it doesn't provide:**
- Module discovery (loader's job)
- Module sources (resolver's job)
- Business logic (modules' job)

## Key Distinctions

### Profile vs. Mount Plan vs. Session

These are **stages in a pipeline**, not interchangeable terms.

```
1. Profile (YAML + Markdown)
   ↓ ProfileLoader.load_profile()
   ↓
2. Profile Object (Python object)
   ↓ compile_profile_to_mount_plan()
   ↓
3. Mount Plan (Dict)
   ↓ AmplifierSession(config=mount_plan)
   ↓
4. Session (Runtime instance)
   ↓ session.initialize()
   ↓
5. Running Session (Modules loaded)
```

**When debugging:**
- Syntax error? → Check Profile parsing (stage 1)
- Module not found? → Check Mount Plan compilation (stage 3)
- Runtime error? → Check Session execution (stage 5)

**Don't conflate them:** A profile is a specification. A session is an instance.

### DisplaySystem vs. Hooks vs. UI

Three separate layers with distinct responsibilities:

```
User Interface (Application Layer)
  ↑ receives SSE events
  ↑
DisplaySystem (Protocol)
  ↑ methods called by
  ↑
Hooks (Observers)
  ↑ receive events from
  ↑
Orchestrator (Kernel)
```

**DisplaySystem:**
- Protocol (interface) for kernel → app communication
- **Injected FROM application** (not part of kernel)
- Methods like: `display_thinking()`, `display_content_delta()`
- Lives in your application code

**Hooks:**
- Observation points within kernel
- **Registered BY application**
- Receive events, return HookResult
- Bridge orchestrator → DisplaySystem

**UI:**
- Application-layer rendering
- **Lives IN your application**
- Interprets DisplaySystem output
- Displays to end user

**Common confusion:** Expecting DisplaySystem to magically exist. It doesn't - you inject it.

### Source vs. Module vs. Entry Point

Three concepts in the module loading pipeline:

```
Source (Where)
  ↓ "git+https://github.com/..." or "file:///path" or "package-name"
  ↓
Module (What)
  ↓ Python package with mount() function satisfying contract
  ↓
Entry Point (How Discovered)
  ↓ Python's pkg_resources.entry_points mechanism
```

**Source:** WHERE to get a module
- Git repository URL
- Local file path
- PyPI package name

**Module:** WHAT you're getting
- Python code in a package
- Exports `mount()` function
- Satisfies a contract (Tool, Provider, etc.)

**Entry Point:** HOW Python finds it
- Registered in `pyproject.toml`
- Discoverable via `importlib.metadata.entry_points()`
- Allows dynamic loading

**The flow:**
1. Profile specifies source: `source: git+https://...`
2. ModuleSourceResolver fetches and caches
3. Module registered via entry point
4. ModuleLoader discovers via entry points
5. Loader imports and calls `mount()`

## Thinking in Layers

Amplifier is layered like a network stack:

```
┌─────────────────────────────────────┐
│ Application Layer                   │ ← Your code
│ (UI, Business Logic, UX Systems)   │
├─────────────────────────────────────┤
│ Kernel Layer                        │ ← amplifier-core
│ (Coordinator, Hooks, Session)      │
├─────────────────────────────────────┤
│ Module Layer                        │ ← Orchestrators, Tools, etc.
│ (Plugins providing capabilities)    │
├─────────────────────────────────────┤
│ Provider Layer                      │ ← LLM APIs
│ (External services)                 │
└─────────────────────────────────────┘
```

**Layer responsibilities:**

**Application Layer (Your Code):**
- User interface and experience
- Business logic and policies
- UX systems (DisplaySystem, ApprovalSystem)
- Integration with your app's architecture

**Kernel Layer (amplifier-core):**
- Module coordination
- Event emission and hook chain
- Session lifecycle
- Boundary enforcement
- **No policies, only mechanisms**

**Module Layer (Plugins):**
- Implement specific capabilities
- Satisfy contracts
- Compose to provide behavior
- Can be swapped without kernel changes

**Provider Layer (External):**
- LLM APIs (Anthropic, OpenAI, etc.)
- External services
- Called by modules, not directly by kernel

**Problem-solving by layer:**

| Symptom | Likely Layer | Investigation |
|---------|--------------|---------------|
| Wrong LLM output | Provider | Check model, prompts, parameters |
| Tool not executing | Module | Check tool implementation, registration |
| Events not flowing | Kernel | Check hooks registration, coordinator |
| UI not updating | Application | Check DisplaySystem, SSE, frontend |

**Don't skip layers:** Trace problems through the stack systematically.

## Understanding Events

Events are the nervous system of Amplifier.

**Mental model:** Pub/Sub messaging

```
Producer (Orchestrator)
  ↓ emits "content_block:delta"
  ↓
Event Bus (Coordinator.hooks)
  ↓ delivers to all registered observers
  ↓
Consumers (Hooks)
  ↓ Hook₁: logs event
  ↓ Hook₂: calls DisplaySystem
  ↓ Hook₃: updates metrics
```

**Event naming convention:**
- `namespace:action` pattern
- Examples: `content_block:delta`, `tool:pre`, `session:start`
- Wildcard: `*` matches all events

**Event flow:**
1. Orchestrator emits: `coordinator.hooks.emit('event:name', data)`
2. Coordinator delivers to registered hooks
3. Each hook processes: `hook(event, data) → HookResult`
4. Chain continues if `HookResult(action='continue')`

**Key insight:** Events are **declarative**. They describe what happened, not what to do.

## Understanding Streaming

Streaming is often misunderstood. Here's the correct mental model:

**Streaming ≠ Magic Real-Time Updates**

Streaming is: **Progressive event emission during execution**

```
Non-streaming:
session.execute(prompt) → [BLACK BOX] → complete response

Streaming:
session.execute(prompt) → content_block:delta → delta → delta → complete
                        ↓
                     events you can observe
```

**What enables streaming:**
1. **Orchestrator** emits delta events (loop-streaming does this)
2. **Hooks** observe the events (you register these)
3. **DisplaySystem** receives calls from hooks (you implement this)
4. **UI** renders progressive updates (your application)

**What doesn't enable streaming:**
- ❌ A flag in config
- ❌ hooks-streaming-ui (it prints to console)
- ❌ Magic in the kernel

**To get streaming working:**
1. Use streaming-capable orchestrator (`loop-streaming`)
2. Register hooks to bridge events → DisplaySystem
3. Implement DisplaySystem to forward to UI
4. UI renders updates progressively

## Common Misconceptions

### "The kernel does X"

**Misconception:** The kernel implements features like logging, formatting, streaming display.

**Reality:** The kernel provides **mechanisms** (hooks, events). Features are **modules**.

**Example:**
- ❌ "The kernel logs events"
- ✅ "The kernel emits events, hooks-logging logs them"

### "I need to configure Y"

**Misconception:** Amplifier has config flags for different behaviors.

**Reality:** Amplifier uses **composition**. Swap modules, don't toggle flags.

**Example:**
- ❌ "How do I enable streaming?" (looking for flag)
- ✅ "Which orchestrator provides streaming?" (looking for module)

### "DisplaySystem is built-in"

**Misconception:** DisplaySystem is part of amplifier-core.

**Reality:** DisplaySystem is a **protocol** you implement and inject.

**Example:**
- ❌ "Why isn't DisplaySystem being called?" (expecting it to exist)
- ✅ "Did I inject DisplaySystem into the session?" (understanding ownership)

### "Hooks modify behavior"

**Misconception:** Hooks change what the system does.

**Reality:** Hooks **observe** what happens. They don't change it.

**Example:**
- ❌ "Use a hook to retry failed tools"
- ✅ "Hooks observe, orchestrator implements retry logic"

## Mental Model Summary

**When you see code, think:**

- `AmplifierSession` → Coordination context, not agent
- `mount()` → Plugging a module into a socket
- `coordinator.hooks.register()` → Adding an observer
- `DisplaySystem` → Protocol I implement, not kernel feature
- `orchestrator: loop-streaming` → Composing a component
- Events → Observations, not commands
- Profile → Blueprint, not instance

**When debugging, trace through layers:**

```
My UI doesn't show X
  ↓ Is DisplaySystem being called?
  ↓ Are hooks calling DisplaySystem?
  ↓ Are events being emitted?
  ↓ Is orchestrator configured correctly?
```

**When designing, think composition:**

```
I need behavior X
  ↓ Is there a module for this?
  ↓ Can I implement it as a module?
  ↓ Can I compose existing modules?
  ↓ Only then: Do I need kernel changes?
```

This mental model makes Amplifier's design intuitive rather than surprising.
