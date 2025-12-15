# Amplifier Dev Collection

A collection for building Amplifier modules, profiles, and collections with systematic thinking patterns. This is the "develop Amplifier stuff" collection that knows:

- What a module is
- How the core/kernel works
- How to make modules (tools, providers, hooks, orchestrators, context managers)
- How to validate and test modules
- How to design profiles and collections
- Evidence-driven reasoning patterns
- Systematic problem-solving approaches
- Session debugging and recovery

## Installation

```bash
amplifier collection add file:///path/to/amplifier-collection-amplifier-dev
# or
amplifier collection add git+https://github.com/microsoft/amplifier-collection-amplifier-dev@main
```

## Usage

```bash
# Use the amplifier-builder profile (recommended for development)
amplifier profile use amplifier-dev:amplifier-builder

# Or use amplifier-dev for quick reference questions
amplifier profile use amplifier-dev:amplifier-dev

# Ask questions about Amplifier internals
amplifier> "How do I create a new tool module?"
amplifier> "Build me a weather tool with proper contract compliance"
amplifier> "Help me design a profile for code review"
```

## What's Included

### Profiles

| Profile | Purpose | When to Use |
|---------|---------|-------------|
| **amplifier-dev** | Amplifier domain knowledge | Learning internals, understanding contracts, quick reference |
| **reasoning-dev** | Systematic reasoning patterns | Complex debugging, evidence-driven investigation (any project) |
| **reasoning-dev-openai** | Reasoning with OpenAI | Same as reasoning-dev, uses OpenAI provider |
| **amplifier-builder** | Combined: domain + reasoning | **Building Amplifier modules** (recommended for development) |

### Agents

- **module-author** - Helps scaffold new modules with correct contracts
- **profile-designer** - Helps design and debug profiles and collections
- **amplifier-explainer** - Explains Amplifier architecture, contracts, and philosophy
- **session-debugger** - Diagnoses and repairs corrupted Amplifier session transcripts

### Context

This collection includes:

**Amplifier-specific documentation:**
- `context/architecture/` - Dynamic references to authoritative docs from `amplifier-core`
- `context/contracts/` - Module contract summaries (Provider, Tool, Hook)
- `context/examples/` - Working code examples
- `context/frameworks/` - Meta-cognitive frameworks for Amplifier development

**Systematic thinking patterns:**
- `context/philosophy/` - THINKING-PHILOSOPHY.md, REASONING-LOOP.md
- `context/protocols/` - PATTERNS.md, ANTI-PATTERNS.md, VELOCITY-PATTERNS.md, WORKFLOW-PATTERNS.md, UNBLOCKING-PLAYBOOK.md, THINKING-PLAYBOOK.md, SESSION-DEBUGGING.md

The collection **dynamically references** authoritative docs from `amplifier-core` rather than copying them, ensuring you always have up-to-date contract specifications.

## Design Decision: Dynamic References

Instead of copying documentation that could become stale, this collection:

1. **Fetches from GitHub** - The profile instructs the agent to use `tool-web` to fetch contracts from `https://raw.githubusercontent.com/microsoft/amplifier-core/main/docs/`
2. **Embedded fallback** - Contract summaries in `context/contracts/` for offline use
3. **Philosophy from foundation** - References `@foundation:context/` for kernel philosophy docs

This ensures the "Develop Amplifier" assistant always has access to the **source of truth** documentation without requiring users to have `amplifier-core` locally.

## Testing Locally

### 1. Register the collection

Add to `.amplifier/settings.yaml`:

```yaml
collection_sources:
  amplifier-dev: ./amplifier-collection-amplifier-dev
```

### 2. Run the profiles

```bash
# Use amplifier-builder for full development capabilities
amplifier run --profile amplifier-dev:amplifier-builder "Build me a weather tool"

# Or use amplifier-dev for quick questions
amplifier run --profile amplifier-dev:amplifier-dev "How do I create a new tool?"

# Or use reasoning-dev for systematic debugging (any project)
amplifier run --profile amplifier-dev:reasoning-dev "Debug this complex issue..."
```

### 3. Run test cases

See `tests/README.md` for the full test suite. Quick smoke test:

```bash
# Test contract fetching (should use tool-web)
amplifier run --profile amplifier-dev:amplifier-builder \
  "What is the Tool contract? Show me the required interface."

# Test module scaffolding with systematic reasoning
amplifier run --profile amplifier-dev:amplifier-builder \
  "Help me create a tool that fetches weather data."
```
