# Tool Contract Summary

> **Authoritative source**: https://raw.githubusercontent.com/microsoft/amplifier-core/main/docs/contracts/TOOL_CONTRACT.md

## Purpose

Tools extend agent capabilities beyond conversation: filesystem ops, command execution, web access, task delegation.

## Protocol

```python
@runtime_checkable
class Tool(Protocol):
    @property
    def name(self) -> str:
        """Tool name for invocation (snake_case, unique)."""
        ...

    @property
    def description(self) -> str:
        """Human-readable description for LLM."""
        ...

    async def execute(self, input: dict[str, Any]) -> ToolResult:
        """Execute tool and return result."""
        ...
```

## Data Models

```python
class ToolCall(BaseModel):
    id: str                    # Unique ID for correlation
    name: str                  # Tool name to invoke
    arguments: dict[str, Any]  # Tool-specific parameters

class ToolResult(BaseModel):
    tool_call_id: str          # Correlates with ToolCall.id
    output: Any                # Tool output
    is_error: bool = False     # Whether execution failed
```

## Implementation Pattern

```python
class MyTool:
    def __init__(self, config: dict):
        self.config = config

    @property
    def name(self) -> str:
        return "my_tool"

    @property
    def description(self) -> str:
        return "Does X with Y parameters."

    async def execute(self, input: dict[str, Any]) -> ToolResult:
        try:
            result = await self._do_work(input)
            return ToolResult(
                tool_call_id=input.get("_tool_call_id", ""),
                output=result,
                is_error=False
            )
        except Exception as e:
            return ToolResult(
                tool_call_id=input.get("_tool_call_id", ""),
                output=f"Error: {e}",
                is_error=True
            )

async def mount(coordinator, config: dict):
    tool = MyTool(config)
    await coordinator.mount("tools", tool, name="my_tool")
    return None  # or cleanup function
```

## Checklist

- [ ] Implements `name`, `description`, `execute()`
- [ ] `mount()` function with entry point
- [ ] Returns `ToolResult` (never raises)
- [ ] Handles errors gracefully (`is_error=True`)
