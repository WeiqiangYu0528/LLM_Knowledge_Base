# Tool Call Execution and Approval Pipeline

## Overview

Hermes' tool pipeline is where abstract model intent becomes concrete side effects. This page follows a single tool call from model output through tool resolution, registry dispatch, environment execution, optional human approval, and result reinsertion into the conversation. It is one of the most important cross-subsystem behaviors in the repo because it touches the agent core, tool runtime, execution backends, and surface-specific approval UX.

## Systems Involved

- [Agent Loop Runtime](../entities/agent-loop-runtime.md)
- [Tool Registry and Dispatch](../entities/tool-registry-and-dispatch.md)
- [Terminal and Execution Environments](../entities/terminal-and-execution-environments.md)
- [Gateway Runtime](../entities/gateway-runtime.md)
- [ACP Adapter](../entities/acp-adapter.md)

## Interaction Model

The normal path is:

1. The model returns one or more tool calls.
2. `AIAgent` parses those calls and decides whether they are agent-level exceptions or general registry dispatches.
3. For a general tool, `model_tools.handle_function_call(...)` hands control to the registry.
4. Plugin pre-hooks may observe the call.
5. If the tool is approval-sensitive, Hermes checks `DANGEROUS_PATTERNS`.
6. When approval is needed, the decision is transported through the active surface:
   - CLI prompt
   - gateway in-band approval
   - ACP permission request
7. The tool handler executes, often through a selected environment backend.
8. Plugin post-hooks may observe the result.
9. Hermes appends a `tool` message with the normalized result and returns control to the agent loop.

The agent-level exceptions (`todo`, `memory`, `session_search`, `delegate_task`) skip part of this path because they need direct access to agent-local structures or subagent machinery.

## Key Interfaces

| Interface | Why it matters |
|------|------|
| `handle_function_call(...)` in `model_tools.py` | Entry point from model tool calls into execution |
| `ToolRegistry.dispatch(...)` | Core handler resolution and exception wrapping |
| `DANGEROUS_PATTERNS` in `tools/approval.py` | Policy trigger for dangerous actions |
| `BaseEnvironment.execute(...)` | Common command-execution contract across backends |
| surface approval callbacks | Transport layer for the same approval decision |

The real boundary shift happens at step 5. Up to that point, the tool pipeline is still within the runtime's control. Once approval is needed, the decision path crosses into whichever shell currently hosts the conversation.

## Source Evidence

- `model_tools.py` — tool discovery and `handle_function_call(...)`
- `tools/registry.py` — dispatch and error wrapping
- `tools/approval.py` — dangerous-command detection
- `tools/terminal_tool.py` and `tools/environments/*` — terminal execution path
- `acp_adapter/permissions.py`, `gateway/run.py`, CLI callbacks — surface-specific approval transports

## See Also

- [Tool Registry and Dispatch](../entities/tool-registry-and-dispatch.md)
- [Terminal and Execution Environments](../entities/terminal-and-execution-environments.md)
- [Interruption and Human Approval Flow](../concepts/interruption-and-human-approval-flow.md)
- [Gateway Message to Agent Reply Flow](gateway-message-to-agent-reply-flow.md)
