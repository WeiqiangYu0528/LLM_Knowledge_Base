# Toolset-Based Capability Governance

## Overview

Hermes governs capability exposure through toolsets, readiness checks, and per-surface presets instead of handing every tool to every session. This gives the repo one tool runtime with many shapes: broad CLI sessions, narrow ACP sessions, platform-specific gateway sessions, and automation-safe cron sessions.

## Mechanism

The governance path works like this:

1. Tool modules self-register with names, schemas, handlers, and optional availability checks.
2. Sessions choose enabled and disabled toolsets based on config or the surface being launched.
3. `model_tools.get_tool_definitions(...)` expands toolsets into concrete tool names.
4. `ToolRegistry.get_definitions(...)` filters those names by `check_fn` availability.
5. Dynamic schema patching updates certain tools so they only reference the tools that actually survived filtering.

The result is not merely "less clutter". The model literally cannot call tools that are absent from the filtered schema list.

## Involved Entities

- [Tool Registry and Dispatch](../entities/tool-registry-and-dispatch.md)
- [CLI Runtime](../entities/cli-runtime.md)
- [ACP Adapter](../entities/acp-adapter.md)
- [Cron System](../entities/cron-system.md)
- [Skills System](../entities/skills-system.md)

## Source Evidence

- `model_tools.py` — tool discovery, toolset resolution, schema filtering
- `tools/registry.py` — availability checks and final schema generation
- `toolsets.py` — named toolset definitions
- `acp_adapter/server.py` — ACP-specific tool-surface refresh logic
- `website/docs/developer-guide/tools-runtime.md` — description of toolset filtering and dynamic patching

## See Also

- [Tool Registry and Dispatch](../entities/tool-registry-and-dispatch.md)
- [Skills System](../entities/skills-system.md)
- [Tool Call Execution and Approval Pipeline](../syntheses/tool-call-execution-and-approval-pipeline.md)
- [CLI Runtime](../entities/cli-runtime.md)
