# Hook Contract Summary

> **Authoritative source**: https://raw.githubusercontent.com/microsoft/amplifier-core/main/docs/contracts/HOOK_CONTRACT.md

## Purpose

Hooks observe, control, or inject into agent cognition at specific lifecycle points.

## Protocol

```python
@runtime_checkable
class Hook(Protocol):
    @property
    def name(self) -> str:
        """Hook identifier."""
        ...

    async def __call__(self, event: str, data: dict[str, Any]) -> HookResult:
        """
        Process event and return action.

        Args:
            event: Event name (e.g., "tool:pre", "session:start")
            data: Event-specific data

        Returns:
            HookResult with action and optional modifications
        """
        ...
```

## HookResult

```python
class HookResult(BaseModel):
    action: str  # "continue" | "block" | "skip"
    reason: str | None = None  # Why blocked/skipped
    inject: str | None = None  # Content to inject into context
    data: dict | None = None   # Modified event data
```

## Standard Events

| Event | When | Data |
|-------|------|------|
| `session:start` | Session begins | `{session_id, config}` |
| `session:end` | Session ends | `{session_id}` |
| `turn:start` | Turn begins | `{prompt}` |
| `turn:end` | Turn ends | `{response}` |
| `provider:pre` | Before LLM call | `{request}` |
| `provider:post` | After LLM call | `{request, response}` |
| `tool:pre` | Before tool execution | `{tool_name, input}` |
| `tool:post` | After tool execution | `{tool_name, input, result}` |

## Implementation Pattern

```python
class ApprovalHook:
    def __init__(self, config: dict):
        self.require_approval = config.get("tools", [])

    @property
    def name(self) -> str:
        return "approval"

    async def __call__(self, event: str, data: dict) -> HookResult:
        if event == "tool:pre":
            tool_name = data.get("tool_name")
            if tool_name in self.require_approval:
                approved = await self._prompt_user(tool_name, data)
                if not approved:
                    return HookResult(
                        action="block",
                        reason=f"User denied {tool_name}"
                    )

        return HookResult(action="continue")

async def mount(coordinator, config: dict):
    hook = ApprovalHook(config)
    await coordinator.mount("hooks", hook, name="approval")
    return None
```

## Hook Actions

| Action | Effect |
|--------|--------|
| `continue` | Proceed normally |
| `block` | Stop the operation |
| `skip` | Skip this operation |

## Use Cases

- **Observability**: Log events, track metrics
- **Control**: Approval gates, rate limiting
- **Injection**: Add context, modify prompts
- **Security**: PII redaction, secret detection

## Checklist

- [ ] Implements `name` and `__call__`
- [ ] Returns `HookResult` (never raises)
- [ ] Handles unknown events gracefully (`action="continue"`)
- [ ] Fast execution (hooks run synchronously in flow)
