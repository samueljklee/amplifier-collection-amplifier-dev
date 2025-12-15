# Example: Creating a Tool Module

This example shows a complete tool implementation following the contract.

## Directory Structure

```
amplifier-module-tool-example/
├── amplifier_module_tool_example/
│   └── __init__.py
├── tests/
│   └── test_example.py
├── pyproject.toml
└── README.md
```

## pyproject.toml

```toml
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"

[project]
name = "amplifier-module-tool-example"
version = "0.1.0"
description = "Example tool that demonstrates the tool contract"
requires-python = ">=3.11"
dependencies = [
    "amplifier-core>=0.1.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "pytest-asyncio>=0.21",
]

[project.entry-points."amplifier.modules"]
tool-example = "amplifier_module_tool_example:mount"
```

## __init__.py

```python
"""
Example Tool - Demonstrates the Amplifier tool contract.

This tool performs a simple operation to show the required interface.
"""

from dataclasses import dataclass
from typing import Any


@dataclass
class ToolResult:
    """Result from tool execution."""
    output: Any
    error: str | None = None


class ExampleTool:
    """
    Example tool implementation.

    Demonstrates:
    - Required properties (name, description)
    - The execute() method signature
    - Configuration handling
    - Error handling patterns
    """

    def __init__(self, config: dict):
        """
        Initialize the tool with configuration.

        Args:
            config: Configuration from the profile, e.g.:
                tools:
                  - module: tool-example
                    config:
                      prefix: "[Example]"
                      uppercase: false
        """
        self.prefix = config.get("prefix", "[Tool]")
        self.uppercase = config.get("uppercase", False)

    @property
    def name(self) -> str:
        """Unique identifier for this tool."""
        return "example"

    @property
    def description(self) -> str:
        """
        Human-readable description shown to the LLM.

        This helps the model understand when to use this tool.
        Be specific about:
        - What the tool does
        - What inputs it expects
        - What outputs it produces
        """
        return (
            "Formats a message with an optional prefix. "
            "Input: {'message': 'text to format'}. "
            "Returns the formatted message."
        )

    @property
    def input_schema(self) -> dict:
        """
        JSON Schema for the tool's input.

        The LLM uses this to construct valid tool calls.
        """
        return {
            "type": "object",
            "properties": {
                "message": {
                    "type": "string",
                    "description": "The message to format"
                }
            },
            "required": ["message"]
        }

    async def execute(self, input: dict) -> ToolResult:
        """
        Execute the tool with the given input.

        Args:
            input: Dictionary matching input_schema

        Returns:
            ToolResult with output or error
        """
        try:
            message = input.get("message")
            if not message:
                return ToolResult(
                    output=None,
                    error="Missing required input: message"
                )

            # Process the message
            result = f"{self.prefix} {message}"
            if self.uppercase:
                result = result.upper()

            return ToolResult(output=result, error=None)

        except Exception as e:
            # Always return ToolResult, never raise
            return ToolResult(output=None, error=str(e))


async def mount(coordinator, config: dict):
    """
    Mount this tool into an Amplifier session.

    This is the entry point called by the kernel during session initialization.

    Args:
        coordinator: The ModuleCoordinator for registration
        config: Tool configuration from the profile

    Returns:
        Optional cleanup function called when session ends
    """
    tool = ExampleTool(config)

    # Register with the coordinator
    # "tools" is the registry name, "example" is the tool name
    await coordinator.mount("tools", tool, name="example")

    # Return optional cleanup function
    async def cleanup():
        # Perform any cleanup needed
        # (this simple tool has nothing to clean up)
        pass

    return cleanup
```

## tests/test_example.py

```python
"""Tests for the example tool."""

import pytest
from amplifier_module_tool_example import ExampleTool, mount


class TestExampleTool:
    """Unit tests for ExampleTool."""

    def test_has_required_properties(self):
        """Tool must have name and description."""
        tool = ExampleTool({})
        assert tool.name == "example"
        assert len(tool.description) > 0

    def test_has_input_schema(self):
        """Tool should define input schema."""
        tool = ExampleTool({})
        schema = tool.input_schema
        assert schema["type"] == "object"
        assert "message" in schema["properties"]

    @pytest.mark.asyncio
    async def test_execute_success(self):
        """Tool returns formatted message."""
        tool = ExampleTool({"prefix": "[Test]"})
        result = await tool.execute({"message": "hello"})
        assert result.error is None
        assert result.output == "[Test] hello"

    @pytest.mark.asyncio
    async def test_execute_with_uppercase(self):
        """Tool respects uppercase config."""
        tool = ExampleTool({"prefix": "[Test]", "uppercase": True})
        result = await tool.execute({"message": "hello"})
        assert result.output == "[TEST] HELLO"

    @pytest.mark.asyncio
    async def test_execute_missing_message(self):
        """Tool handles missing input gracefully."""
        tool = ExampleTool({})
        result = await tool.execute({})
        assert result.error is not None
        assert "message" in result.error.lower()

    @pytest.mark.asyncio
    async def test_execute_never_raises(self):
        """Tool should return error, not raise."""
        tool = ExampleTool({})
        # Even with bad input, should return ToolResult
        result = await tool.execute(None)  # type: ignore
        assert result.error is not None


class TestMount:
    """Tests for the mount function."""

    @pytest.mark.asyncio
    async def test_mount_registers_tool(self):
        """Mount should register tool with coordinator."""

        class MockCoordinator:
            def __init__(self):
                self.mounted = []

            async def mount(self, registry, instance, name):
                self.mounted.append((registry, name))

        coordinator = MockCoordinator()
        cleanup = await mount(coordinator, {})

        assert ("tools", "example") in coordinator.mounted
        assert cleanup is not None

    @pytest.mark.asyncio
    async def test_cleanup_callable(self):
        """Cleanup function should be callable."""

        class MockCoordinator:
            async def mount(self, *args, **kwargs):
                pass

        cleanup = await mount(MockCoordinator(), {})
        await cleanup()  # Should not raise
```

## Usage in Profile

```yaml
---
profile:
  name: test-example
  extends: base

tools:
  - module: tool-example
    source: file:///path/to/amplifier-module-tool-example
    config:
      prefix: "[Demo]"
      uppercase: true
---

You have access to an example formatting tool.
```

## Key Takeaways

1. **Properties over methods** for name/description - cleaner interface
2. **input_schema** helps the LLM construct valid calls
3. **execute() returns ToolResult** - never raise exceptions
4. **mount() is the entry point** - register and return cleanup
5. **Config comes from profile** - design for configurability
6. **Tests validate the contract** - ensure compliance
