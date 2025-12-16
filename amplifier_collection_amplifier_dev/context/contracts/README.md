# Contract Summaries (Embedded Fallback)

These are **snapshots** of the Amplifier module contracts for offline use.

**For the authoritative, up-to-date versions, fetch from GitHub:**
```
https://raw.githubusercontent.com/microsoft/amplifier-core/main/docs/contracts/
```

Last synced: 2025-01-29

## Quick Reference

| Module Type | Key Interface | Entry Point |
|-------------|---------------|-------------|
| Provider | `complete(request) -> ChatResponse` | `mount(coordinator, config)` |
| Tool | `execute(input) -> ToolResult` | `mount(coordinator, config)` |
| Hook | `__call__(event, data) -> HookResult` | `mount(coordinator, config)` |
| Orchestrator | `execute(prompt, context, ...) -> str` | `mount(coordinator, config)` |
| Context | `add_message()`, `get_messages()` | `mount(coordinator, config)` |

## Common Pattern: mount() Function

All modules follow the same entry point pattern:

```python
async def mount(coordinator: ModuleCoordinator, config: dict):
    """
    Initialize and register the module.

    Args:
        coordinator: For registration via coordinator.mount()
        config: Configuration from profile

    Returns:
        Optional cleanup function (called on session end)
    """
    instance = MyModule(config)
    await coordinator.mount("{type}s", instance, name="{name}")

    async def cleanup():
        await instance.close()
    return cleanup
```

```toml
# pyproject.toml
[project.entry-points."amplifier.modules"]
{type}-{name} = "amplifier_module_{type}_{name}:mount"
```
