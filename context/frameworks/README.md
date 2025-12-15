# Amplifier Development Frameworks

## Purpose

This directory contains **meta-cognitive frameworks** for Amplifier development. These documents teach you HOW TO THINK about Amplifier problems, not what specific solutions to apply.

## What's Here

| Framework | Purpose | Use When |
|-----------|---------|----------|
| [DIAGNOSTIC_METHODOLOGY](DIAGNOSTIC_METHODOLOGY.md) | Systematic debugging approach | Integration fails, unclear why |
| [ARCHITECTURE_REASONING](ARCHITECTURE_REASONING.md) | Understanding Amplifier's design | Architecture decisions, design questions |
| [PROBLEM_SOLVING_HEURISTICS](PROBLEM_SOLVING_HEURISTICS.md) | Mental shortcuts for common issues | Stuck on implementation, need guidance |
| [CONCEPTUAL_MODEL](CONCEPTUAL_MODEL.md) | Mental model of how Amplifier works | Learning Amplifier, explaining to others |
| [INVESTIGATION_PROTOCOL](INVESTIGATION_PROTOCOL.md) | Step-by-step investigation process | Systematic problem investigation |
| [DESIGN_PRINCIPLES](DESIGN_PRINCIPLES.md) | Core principles guiding decisions | Design decisions, evaluating solutions |

## How to Use These Frameworks

### For Learning Amplifier

**Start here:**
1. [CONCEPTUAL_MODEL](CONCEPTUAL_MODEL.md) - Build mental model
2. [ARCHITECTURE_REASONING](ARCHITECTURE_REASONING.md) - Understand design philosophy
3. [DESIGN_PRINCIPLES](DESIGN_PRINCIPLES.md) - Learn guiding principles

### For Debugging Issues

**Start here:**
1. [DIAGNOSTIC_METHODOLOGY](DIAGNOSTIC_METHODOLOGY.md) - Systematic diagnosis
2. [INVESTIGATION_PROTOCOL](INVESTIGATION_PROTOCOL.md) - Step-by-step process
3. [PROBLEM_SOLVING_HEURISTICS](PROBLEM_SOLVING_HEURISTICS.md) - Quick patterns

### For Building Integrations

**Start here:**
1. [DESIGN_PRINCIPLES](DESIGN_PRINCIPLES.md) - Guiding principles
2. [ARCHITECTURE_REASONING](ARCHITECTURE_REASONING.md) - Thinking frameworks
3. [PROBLEM_SOLVING_HEURISTICS](PROBLEM_SOLVING_HEURISTICS.md) - Implementation shortcuts

## Philosophy

These frameworks embody several key ideas:

### Transferable Knowledge

Rather than documenting specific solutions (which become outdated), we capture the **reasoning patterns** that lead to solutions. These patterns apply across scenarios.

### Systematic Over Random

Effective debugging and development isn't random trial and error. It's systematic application of frameworks and heuristics.

### Mental Models Matter

Understanding HOW Amplifier is designed (and why) makes everything else intuitive. The right mental model prevents confusion.

### Learn Once, Apply Everywhere

These frameworks aren't Amplifier-specific tricks. They're general problem-solving approaches applied to Amplifier development.

## Relationship to Other Documentation

**Foundation collection** provides:
- KERNEL_PHILOSOPHY.md - Why Amplifier is designed this way
- IMPLEMENTATION_PHILOSOPHY.md - General software development philosophy
- MODULAR_DESIGN_PHILOSOPHY.md - Bricks-and-studs approach

**These frameworks** provide:
- How to THINK about Amplifier problems
- How to DIAGNOSE Amplifier issues
- How to REASON about Amplifier architecture
- How to APPLY the philosophy practically

**Contracts** (in `../contracts/`) provide:
- What modules MUST do (specifications)
- What kernel GUARANTEES (invariants)

**Examples** (in `../examples/`) provide:
- Working CODE demonstrating patterns

Together, they form a complete knowledge base:
```
Philosophy → Frameworks → Contracts → Examples → Your Code
   ↓            ↓           ↓           ↓           ↓
  Why?       How to       What's      Show me    I build
            think?       required?   working
```

## Contributing to These Frameworks

When you solve a challenging Amplifier problem, consider:

1. **What reasoning process led to the solution?**
   - Could this process apply to other problems?
   - Is it documented in these frameworks?

2. **What mental model helped?**
   - Is this model in CONCEPTUAL_MODEL.md?
   - Would explaining it help others?

3. **What heuristic would have saved time?**
   - Could this pattern be generalized?
   - Should it be in PROBLEM_SOLVING_HEURISTICS.md?

The goal: Capture the THINKING PROCESS, not the specific solution.

## Example Usage

**Scenario:** Integration failing, DisplaySystem methods not called

**Framework Application:**

1. **DIAGNOSTIC_METHODOLOGY:** Work from kernel outward
   - Layer 1: Is kernel set up? ✅
   - Layer 2: Are modules registered? ✅
   - Layer 3: Are events flowing? ✅
   - Layer 4: Is DisplaySystem being called? ❌ ← Found the layer

2. **INVESTIGATION_PROTOCOL:** Systematic investigation
   - Expected: DisplaySystem called
   - Actual: Not called
   - Gap: Events fire but DisplaySystem not called
   - Hypothesis: Missing bridge between events and DisplaySystem
   - Test: Check if hooks call DisplaySystem
   - Result: hooks-streaming-ui prints, doesn't call DisplaySystem

3. **ARCHITECTURE_REASONING:** Understand design
   - DisplaySystem is injected FROM app (not part of kernel)
   - Hooks observe events (observability mechanism)
   - Need bridge hook to connect orchestrator → DisplaySystem

4. **PROBLEM_SOLVING_HEURISTICS:** Apply patterns
   - "Work from known-good outward" - Check how CLI does it
   - "Read the contracts" - DisplaySystem protocol vs hooks-streaming-ui
   - "Simplify to minimal" - Create minimal bridge hook

**Outcome:** Solution found through systematic application of frameworks, not random guessing.

## Updating These Frameworks

These frameworks should evolve as we learn:

- **Add new heuristics** when patterns emerge
- **Refine conceptual models** when understanding deepens
- **Document new diagnostic patterns** when common issues arise
- **Update principles** when philosophy evolves

But always: Capture THINKING PATTERNS, not specific solutions.

## For Agent Developers

When creating agents that work with Amplifier:

**Include these frameworks in context** so agents can:
- Reason about Amplifier architecture correctly
- Diagnose issues systematically
- Apply appropriate heuristics
- Maintain philosophical alignment

**Don't include:**
- Specific code examples (they go stale)
- Hardcoded solutions (they're context-specific)
- Step-by-step recipes (they don't generalize)

**Do include:**
- These frameworks (they're timeless)
- Contracts (they're stable)
- Philosophy (it's foundational)

The goal: Agents that THINK correctly about Amplifier, not agents that MEMORIZE solutions.
