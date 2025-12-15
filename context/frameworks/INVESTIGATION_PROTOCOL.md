# Systematic Investigation Protocol

## Purpose

This protocol provides a structured approach to investigating failures in Amplifier integrations. It prevents random debugging and ensures systematic progress toward solutions.

## The Six-Phase Protocol

### Phase 1: Define Expected Behavior

Before investigating any failure, clearly define what success looks like.

**Questions to answer:**
- What should happen?
- Based on what documentation/contract?
- What observable evidence would prove it works?
- What's the simplest test of success?

**Template:**
```
Expected Behavior:
- Action: [what you're doing]
- Expected Result: [what should happen]
- Evidence: [how you'd know it worked]
- Source: [contract/doc that defines this]
```

**Example:**
```
Expected Behavior:
- Action: Create session with vscode-simple profile
- Expected Result: Session initializes, status = "idle"
- Evidence: session.status == "idle", no exceptions
- Source: AmplifierSession contract
```

**Why this matters:** You can't debug if you don't know what "working" looks like.

### Phase 2: Observe Actual Behavior

Gather facts without interpretation. Just observe what IS happening.

**What to collect:**
- Error messages (exact text)
- Events emitted (or not emitted)
- Log output
- Return values
- State at failure point

**Template:**
```
Actual Behavior:
- Error: [exact error message]
- Events: [list of events that fired]
- Logs: [relevant log entries]
- State: [system state at failure]
```

**Example:**
```
Actual Behavior:
- Error: "Module 'loop-streaming' not found via entry points or filesystem"
- Events: None (failed before session initialization)
- Logs: "Loading orchestrator: loop-streaming" → exception
- State: Session created but not initialized
```

**Anti-pattern:** Guessing what's happening instead of observing.

### Phase 3: Identify the Gap

Compare expected vs. actual to find the divergence point.

**Questions to answer:**
- Where do they diverge?
- What assumption broke?
- What prerequisite is missing?
- Which layer failed?

**Template:**
```
Gap Analysis:
- Expected: [what should happen]
- Actual: [what did happen]
- Divergence Point: [where they differ]
- Failed Layer: [which layer failed]
```

**Example:**
```
Gap Analysis:
- Expected: Module loads from git source
- Actual: Module not found
- Divergence Point: Module discovery/loading
- Failed Layer: Module resolution (before kernel)
```

**Why this matters:** The gap points to the root cause.

### Phase 4: Form Hypothesis

Based on the gap, form a **falsifiable** hypothesis about the cause.

**Hypothesis criteria:**
- Explains all observed symptoms
- Is specific and testable
- Makes predictions you can verify
- Simpler than alternative explanations (Occam's Razor)

**Template:**
```
Hypothesis:
- Cause: [what you think is wrong]
- Explains: [which symptoms this accounts for]
- Predicts: [what else should be true if this is correct]
- Test: [how to verify or falsify this]
```

**Example:**
```
Hypothesis:
- Cause: ModuleSourceResolver not mounted before session.initialize()
- Explains: Why git sources aren't resolved
- Predicts: Entry point discovery should work but git sources don't
- Test: Check if resolver is mounted; try mounting it before init
```

**Anti-pattern:** Hypothesis that can't be proven false (unfalsifiable).

### Phase 5: Test Hypothesis

Design a minimal test that proves or disproves the hypothesis.

**Test criteria:**
- Tests ONE thing
- Has clear pass/fail outcome
- Runs quickly
- Isolates the hypothesis

**Template:**
```
Test Design:
- Hypothesis: [what you're testing]
- Test: [minimal code to verify]
- Pass Condition: [what proves it correct]
- Fail Condition: [what proves it wrong]
```

**Example:**
```python
# Test Design:
# Hypothesis: Resolver not mounted
# Test: Check if coordinator has resolver before initialize()
# Pass: Resolver exists → hypothesis FALSE (it is mounted)
# Fail: Resolver missing → hypothesis TRUE (it's not mounted)

session = AmplifierSession(config=mount_plan)
has_resolver = hasattr(session.coordinator, '_module_source_resolver')
print(f"Resolver mounted: {has_resolver}")
```

**Why minimal tests:** Fast feedback loop. Fail fast, learn fast.

### Phase 6: Iterate or Resolve

Based on test results, either solve it or refine your understanding.

**If hypothesis confirmed:**
```
✅ Implement the fix
✅ Verify fix solves original problem
✅ Document the root cause
✅ Update mental model
```

**If hypothesis rejected:**
```
❌ Update your mental model
❌ Form new hypothesis (return to Phase 4)
❌ Gather more observations if needed (return to Phase 2)
❌ Question assumptions (return to Phase 1)
```

**If hypothesis partially confirmed:**
```
⚠️  Refine the hypothesis
⚠️  Design better test
⚠️  Return to Phase 4 with new information
```

**Template:**
```
Result:
- Hypothesis: [confirmed/rejected/partial]
- Next Action: [implement fix / form new hypothesis / gather more data]
- Learning: [what this taught you about the system]
```

## Meta-Principle: Externalize Your Reasoning

Throughout all phases, write down your reasoning. This makes flawed logic visible.

**Investigation Log Template:**
```
Investigation: [problem description]

Phase 1 - Expected:
[what should happen]

Phase 2 - Actual:
[what is happening]

Phase 3 - Gap:
[where they diverge]

Phase 4 - Hypothesis #1:
[theory about cause]

Phase 5 - Test #1:
[how you tested it]
Result: [confirmed/rejected]

Phase 4 - Hypothesis #2:
[revised theory]

...

Resolution:
[root cause]
[fix applied]
[lessons learned]
```

**Why externalize:**
- Catches faulty logic
- Prevents circular reasoning
- Creates shareable documentation
- Helps future debugging

## Common Investigation Patterns

### Pattern: "Works Sometimes, Fails Sometimes"

**Modified protocol:**
1. Define: What's different between working and failing cases?
2. Observe: Capture state in both cases
3. Gap: Find the varying factor
4. Hypothesis: This variable causes the difference
5. Test: Control the variable, verify causation

### Pattern: "Worked Before, Broke Recently"

**Modified protocol:**
1. Define: What changed?
2. Observe: Diff current vs. previous state
3. Gap: Identify the change
4. Hypothesis: This change broke it
5. Test: Revert change, verify fix

### Pattern: "Works in X, Fails in Y"

**Modified protocol:**
1. Define: What's different between X and Y?
2. Observe: Compare environments, configs, versions
3. Gap: Find environmental difference
4. Hypothesis: This difference causes failure
5. Test: Make Y match X, verify it works

## When the Protocol Stalls

If you're cycling through phases without progress:

**Check for:**
- **Wrong assumptions** in Phase 1 (expected behavior is wrong)
- **Incomplete observation** in Phase 2 (missing key data)
- **Wrong gap** in Phase 3 (focusing on wrong divergence)
- **Unfalsifiable hypothesis** in Phase 4 (can't test it)
- **Wrong test** in Phase 5 (testing wrong thing)

**Recovery strategies:**
1. **Start over from Phase 1** with fresh eyes
2. **Get a second opinion** on your hypotheses
3. **Simplify the scenario** to minimal reproduction
4. **Read the source code** if docs aren't helping
5. **Take a break** - come back with fresh perspective

## Protocol Shortcuts (When Appropriate)

For trivial issues, you can compress phases:

**Quick protocol:**
1. What should happen vs. what does happen?
2. What's the obvious cause?
3. Try the obvious fix
4. If it works, done. If not, use full protocol.

**When to use quick protocol:**
- Issue is obviously simple (typo, missing import)
- Fix is low-risk (easily reversible)
- Time pressure justifies risk

**When NOT to use:**
- Issue affects production
- Root cause is unclear
- Multiple competing hypotheses
- Learning is important (not just fixing)

## Example Investigation

**Problem:** DisplaySystem methods never called

**Phase 1 - Expected:**
```
DisplaySystem.display_thinking() should be called when thinking events occur
Source: DisplaySystem protocol, orchestrator should call it
Evidence: Method is called, logs show invocation
```

**Phase 2 - Actual:**
```
display_thinking() never called
Events: content_block:start, content_block:end fire
Hooks: hooks-streaming-ui registered
DisplaySystem: Injected correctly
```

**Phase 3 - Gap:**
```
Events fire → Hooks registered → But DisplaySystem not called
Divergence: Between hooks and DisplaySystem
Failed layer: Hook → DisplaySystem bridge
```

**Phase 4 - Hypothesis:**
```
Cause: hooks-streaming-ui doesn't call DisplaySystem (prints to console)
Explains: Events fire but DisplaySystem not called
Predicts: If I check hooks-streaming-ui source, it won't call DisplaySystem
Test: Read hooks-streaming-ui source code
```

**Phase 5 - Test:**
```python
# Check hooks-streaming-ui source
# Result: It calls print(), not DisplaySystem
# Hypothesis: CONFIRMED
```

**Phase 6 - Resolve:**
```
Root cause: hooks-streaming-ui is for CLI, not programmatic UX
Fix: Create custom hook that bridges orchestrator → DisplaySystem
Result: DisplaySystem now receives calls correctly
Learning: Standard hooks might not fit all use cases
```

## Success Criteria

You've successfully investigated when you can answer:

✅ What was wrong?
✅ Why did it fail?
✅ How did you find it?
✅ How did you fix it?
✅ What did you learn?

If you can't answer all five, keep investigating.
