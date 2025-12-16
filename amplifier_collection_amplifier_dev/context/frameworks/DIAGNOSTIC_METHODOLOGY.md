# Systematic Diagnosis for Amplifier Integration Issues

## Purpose

This framework provides a systematic approach to diagnosing Amplifier integration failures by working from kernel mechanisms outward through the layers of the system.

## The Layered Verification Approach

When encountering integration failures, diagnose from **kernel outward**, not application inward.

### Layer 1: Kernel Mechanisms

Verify the fundamental mechanisms are in place:

**Questions to ask:**
- Is the resolver mounted before session initialization?
- Are collections discoverable in search paths?
- Can profiles compile to mount plans?
- Are module entry points registered?

**Why start here:** Without kernel mechanisms, nothing else can work. These are prerequisites, not symptoms.

### Layer 2: Module Contracts

Verify modules satisfy their contracts:

**Questions to ask:**
- Does the module export a `mount()` function with correct signature?
- Does the mount function register on the correct mount point?
- Does the module return proper types (HookResult, etc.)?
- Are dependencies declared and available?

**Why check here:** Modules that violate contracts fail silently or with cryptic errors.

### Layer 3: Data Flow

Verify data flows through boundaries:

**Questions to ask:**
- Are events being emitted where expected?
- Are hooks receiving the events they're registered for?
- Is data crossing boundaries in the expected format?
- Are coordinators mediating module interactions?

**Why trace flow:** Data stops flowing at integration points - that's where bugs hide.

### Layer 4: Policy/Application Layer

Verify application-layer policies:

**Questions to ask:**
- Are UX systems being called with expected data?
- Is the UI correctly interpreting system outputs?
- Are application-layer decisions being made correctly?
- Is the end-user experience correct?

**Why check last:** This layer depends on all previous layers working.

## Principle: Test Each Layer Independently

Rather than testing the full stack at once:

1. **Isolate the layer** - Can you test it without dependencies?
2. **Verify inputs** - Are inputs to this layer correct?
3. **Verify outputs** - Are outputs from this layer correct?
4. **Only then test integration** - Connect layers after verifying each

**Example:**
```python
# Don't test full stack:
session.execute(prompt) # Black box - where did it fail?

# Instead, test layers:
1. Can you create the session? (kernel)
2. Can you call session.coordinator.hooks.emit()? (module)
3. Do registered hooks receive events? (data flow)
4. Does DisplaySystem get called? (policy)
```

## Principle: Trust But Verify Assumptions

Documentation tells you what SHOULD happen. Code tells you what DOES happen.

**Pattern:**
1. **State assumption:** "Module X should be available"
2. **Design verification:** How can I prove/disprove this?
3. **Execute test:** Run minimal code to check
4. **Observe result:** What actually happened?
5. **Update model:** Revise understanding based on reality

**Anti-pattern:**
- Assuming documentation is current
- Assuming examples reflect actual behavior
- Assuming "it should work" without testing

## Principle: Follow the Data

When things don't work, trace where data stops flowing:

1. **Identify the source** - Where does the data originate?
2. **Trace transformations** - What modifies it along the way?
3. **Find the destination** - Where should it end up?
4. **Locate the break** - Where does it stop flowing?

**Technique: Add observability at boundaries**
```python
# At each boundary, log what crosses:
coordinator.hooks.register('event:name', lambda e, d: print(f"Event: {e}, Data: {d}"))
```

## Principle: Simplify to Minimal Reproduction

Can't figure out why a complex scenario fails?

1. **Strip away everything optional** - Remove all non-essential config
2. **Create minimal example** - Smallest code that should work
3. **Verify the minimal case** - Does it work?
4. **Add complexity incrementally** - One piece at a time
5. **Identify the breaking change** - What addition caused failure?

**Example:**
```python
# Instead of testing full profile with 10 modules:
# Test with just orchestrator + context (minimal viable session)
# Then add providers, tools, hooks one at a time
```

## Principle: Check Prerequisites Before Debugging Behavior

Before debugging "why doesn't X work", verify "is X set up correctly":

**Setup checklist:**
- Are dependencies installed? (`importlib.metadata.entry_points()`)
- Are paths correct? (Collection search paths exist?)
- Are entry points registered? (Can you import the module?)
- Is the environment configured? (Resolver mounted?)

**Why:** 80% of "behavior bugs" are actually "setup bugs"

## Common Diagnostic Patterns

### Pattern: "Module not found"

**Diagnostic sequence:**
1. Check if module is in entry points
2. Check if ModuleSourceResolver is mounted (critical!)
3. Check if profile `source:` field is formatted correctly
4. Check module cache: `~/.amplifier/module-cache/`
5. Test direct resolution: `resolver.resolve(module_id, profile_hint)`

### Pattern: "Session hangs on initialize()"

**Diagnostic sequence:**
1. Is this first-time module download? (git clone can take 30s+)
2. Does orchestrator module resolve?
3. Does context manager module resolve?
4. Are all module sources reachable?
5. Use timeout wrapper: `asyncio.wait_for(session.initialize(), timeout=10)`

### Pattern: "Events not firing"

**Diagnostic sequence:**
1. Are you registering hooks BEFORE session.initialize()?
2. Are you registering the right event name? (exact string match)
3. Does your hook return HookResult(action="continue")?
4. Is the orchestrator actually emitting that event?
5. Hook into `*` to see ALL events: `coordinator.hooks.register('*', debug_handler)`

### Pattern: "DisplaySystem methods not called"

**Diagnostic sequence:**
1. Is DisplaySystem injected into session?
2. Are hooks bridging orchestrator → DisplaySystem?
3. Is hooks-streaming-ui present? (It WON'T call DisplaySystem!)
4. Are events being emitted by orchestrator?
5. Add logging inside DisplaySystem methods to verify

## Meta-Principle: Externalize Your Reasoning

Write down your diagnostic process:

**Template:**
```
Testing: [what you're testing]
Expected: [what should happen]
Observed: [what actually happened]
Conclusion: [what this tells you]
Next: [what to test next]
```

**Why:** Externalizing makes flawed reasoning visible. You'll catch your own mistakes.

## When to Stop Digging

You've found the root cause when:
- ✅ You can reproduce the failure consistently
- ✅ You can explain WHY it fails
- ✅ You can predict what would fix it
- ✅ Fixing it actually fixes the problem

Until then, keep tracing backwards through the layers.
