# Tool Registry and Dispatch

## Overview

Hermes tools are self-registering functions exposed to the model through a central registry. This subsystem is more than "a list of functions": it handles discovery of core tools, conditional availability checks, toolset filtering, dynamic MCP/plugin tool expansion, dispatch of sync and async handlers, and the approval-sensitive terminal path that protects dangerous actions.

## Key Types

| Type or function | Role |
|------|------|
| `ToolRegistry` in `tools/registry.py` | Singleton registry holding tool metadata and handlers |
| `ToolRegistry.register()` | Module-level registration entry point called when tool files import |
| `ToolRegistry.get_definitions()` | Filters tool schemas by tool name and availability checks |
| `ToolRegistry.dispatch()` | Executes a tool handler and wraps failures |
| `_discover_tools()` in `model_tools.py` | Imports tool modules so they self-register |
| `DANGEROUS_PATTERNS` in `tools/approval.py` | Pattern list used by terminal approval gating |

Representative registry API:

```python
class ToolRegistry:
    def register(...): ...
    def get_definitions(self, tool_names: Set[str], quiet: bool = False) -> List[dict]: ...
    def dispatch(self, name: str, args: dict, **kwargs) -> str: ...
```

## Architecture

The runtime is deliberately split into three layers:

1. **Registration layer:** individual tool modules in `tools/` call `registry.register(...)` at import time.
2. **Discovery and schema layer:** `model_tools.py` imports those modules, resolves toolsets, and builds the filtered schema list shown to the model.
3. **Execution layer:** either `AIAgent` intercepts an agent-level tool, or the call flows into `ToolRegistry.dispatch()` and the actual handler.

This design means new tools can often be added by shipping a module that registers itself rather than editing a central switch statement. The cost is that discovery order and import side effects matter, which is why `model_tools.py` is a key runtime owner.

## Runtime Behavior

### Registration and discovery

Core tool modules are loaded by `_discover_tools()` in `model_tools.py`. Importing a module executes its `registry.register()` calls, which populate the singleton registry. After that, Hermes can also discover:

- MCP tools from configured external servers
- plugin-defined tools from the general plugin system

That means the model-facing capability surface is partly static and partly dynamic.

### Availability and toolsets

Registry entries may provide `check_fn` callables. Those are executed during `get_definitions()` and can hide tools when prerequisites are missing. Common reasons for a tool to disappear:

- missing API key
- missing binary or browser dependency
- unavailable external service
- disabled or absent toolset

Toolsets add a second layer of governance. `get_tool_definitions()` resolves enabled and disabled toolsets into concrete tool names, then asks the registry only for those names. This is how Hermes can keep an editor-safe `hermes-acp` surface while exposing a broader CLI or gateway surface elsewhere.

### Dispatch and error handling

At execution time, normal tool calls follow this route:

1. The model emits a tool call.
2. `AIAgent` hands the request to `model_tools.handle_function_call(...)`.
3. Plugin pre-hooks can observe the call.
4. `ToolRegistry.dispatch()` resolves the handler and executes it.
5. Plugin post-hooks can observe the result.
6. The tool result is appended as a `tool` message back into the conversation.

Both `dispatch()` and its higher-level wrapper catch exceptions and convert them into structured JSON error strings. Hermes prefers returning an explicit tool failure to crashing the whole turn loop.

### Agent-level tool exceptions

Four tools are special-cased by the agent loop:

- `todo`
- `memory`
- `session_search`
- `delegate_task`

Their schemas are still part of the tool surface, but the runtime intercepts them before general registry dispatch because they need access to agent-local state, session search handles, or delegation semantics.

### Approval-sensitive execution

The terminal path is where registry dispatch crosses into higher-risk execution. `tools/approval.py` defines `DANGEROUS_PATTERNS` covering operations such as recursive deletes, destructive SQL, filesystem formatting, and suspicious shell pipelines. When the terminal tool detects a match, Hermes routes the decision through a surface-specific approval callback:

- local prompt in CLI
- in-band approval flow in gateway
- ACP permission bridge in editors

This is an architectural boundary, not just a UX nicety: approval transport belongs to the surface, but the policy trigger belongs to the tool runtime.

## Source Files

| File | Purpose |
|------|---------|
| `tools/registry.py` | Registry class, registration, schema filtering, dispatch |
| `model_tools.py` | Tool discovery, toolset resolution, MCP and plugin-tool integration |
| `tools/approval.py` | Dangerous-command detection and approval metadata |
| `tools/terminal_tool.py` | Terminal tool orchestration and environment bridging |
| `website/docs/developer-guide/tools-runtime.md` | Maintainer documentation for registration, toolsets, and dispatch |

## See Also

- [Terminal and Execution Environments](terminal-and-execution-environments.md)
- [Skills System](skills-system.md)
- [Toolset-Based Capability Governance](../concepts/toolset-based-capability-governance.md)
- [Tool Call Execution and Approval Pipeline](../syntheses/tool-call-execution-and-approval-pipeline.md)
