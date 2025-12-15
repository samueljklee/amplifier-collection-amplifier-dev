# Provider Contract Summary

> **Authoritative source**: https://raw.githubusercontent.com/microsoft/amplifier-core/main/docs/contracts/PROVIDER_CONTRACT.md

## Purpose

Providers translate between Amplifier's unified message format and vendor-specific LLM APIs.

## Protocol

```python
@runtime_checkable
class Provider(Protocol):
    @property
    def name(self) -> str:
        """Provider identifier."""
        ...

    def get_info(self) -> ProviderInfo:
        """Return provider metadata."""
        ...

    async def list_models(self) -> list[ModelInfo]:
        """List available models."""
        ...

    async def complete(self, request: ChatRequest, **kwargs) -> ChatResponse:
        """Send request to LLM and return response."""
        ...

    def parse_tool_calls(self, response: ChatResponse) -> list[ToolCall]:
        """Extract tool calls from response."""
        ...
```

## Key Models

```python
class ChatRequest(BaseModel):
    messages: list[Message]
    model: str | None = None
    tools: list[ToolDefinition] | None = None
    max_tokens: int | None = None
    temperature: float | None = None

class ChatResponse(BaseModel):
    id: str
    content: list[ContentBlock]
    model: str
    usage: Usage
    stop_reason: str | None = None

class Usage(BaseModel):
    input_tokens: int
    output_tokens: int
    total_tokens: int
```

## Implementation Pattern

```python
class MyProvider:
    def __init__(self, api_key: str, config: dict):
        self.client = SomeClient(api_key)
        self.config = config

    @property
    def name(self) -> str:
        return "my-provider"

    def get_info(self) -> ProviderInfo:
        return ProviderInfo(
            name=self.name,
            version="1.0.0",
            capabilities=["chat", "tools"]
        )

    async def list_models(self) -> list[ModelInfo]:
        return [ModelInfo(id="model-v1", name="Model V1")]

    async def complete(self, request: ChatRequest, **kwargs) -> ChatResponse:
        # Convert to vendor format, call API, convert back
        vendor_request = self._to_vendor(request)
        vendor_response = await self.client.chat(vendor_request)
        return self._from_vendor(vendor_response)

    def parse_tool_calls(self, response: ChatResponse) -> list[ToolCall]:
        # Extract tool calls from content blocks
        ...

async def mount(coordinator, config: dict):
    api_key = config.get("api_key") or os.environ.get("API_KEY")
    if not api_key:
        return None  # Graceful degradation

    provider = MyProvider(api_key, config)
    await coordinator.mount("providers", provider, name="my-provider")

    async def cleanup():
        await provider.client.close()
    return cleanup
```

## Checklist

- [ ] Implements all 5 protocol methods
- [ ] `mount()` with graceful degradation
- [ ] Preserves content block types (especially `ThinkingBlock.signature`)
- [ ] Reports `Usage` (tokens)
- [ ] Cleanup function for resources
