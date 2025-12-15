# Architecture Context - Dynamic References

This directory explains how the amplifier-dev collection fetches authoritative documentation.

## Strategy: WebFetch + Embedded Fallback

The profile instructs the agent to:

1. **Primary**: Use `tool-web` to fetch contracts from GitHub
2. **Fallback**: Use embedded summaries in `context/contracts/` for offline use

## GitHub URLs (Authoritative Source)

Base: `https://raw.githubusercontent.com/microsoft/amplifier-core/main/docs/`

| Document | Path |
|----------|------|
| PROVIDER_CONTRACT.md | `contracts/PROVIDER_CONTRACT.md` |
| TOOL_CONTRACT.md | `contracts/TOOL_CONTRACT.md` |
| HOOK_CONTRACT.md | `contracts/HOOK_CONTRACT.md` |
| ORCHESTRATOR_CONTRACT.md | `contracts/ORCHESTRATOR_CONTRACT.md` |
| CONTEXT_CONTRACT.md | `contracts/CONTEXT_CONTRACT.md` |
| DESIGN_PHILOSOPHY.md | `DESIGN_PHILOSOPHY.md` |
| MODULE_SOURCE_PROTOCOL.md | `MODULE_SOURCE_PROTOCOL.md` |
| MOUNT_PLAN_SPECIFICATION.md | `specs/MOUNT_PLAN_SPECIFICATION.md` |

## Philosophy Docs

These come from the `foundation` collection (must be installed):
- `@foundation:context/KERNEL_PHILOSOPHY.md`
- `@foundation:context/IMPLEMENTATION_PHILOSOPHY.md`
- `@foundation:context/MODULAR_DESIGN_PHILOSOPHY.md`

## Why Not Copy?

Copying documentation leads to:
- **Stale information** - contracts evolve
- **Drift from source of truth** - users get wrong info
- **Maintenance burden** - must sync manually

WebFetch ensures the agent always gets the latest version.
