# ACP Editor Session Bridge

## Overview

The ACP adapter lets editor clients treat Hermes like an async, session-oriented agent server even though the underlying `AIAgent` runtime is synchronous and callback-driven. This synthesis explains how the bridge works and where the boundaries lie between editor transport, session management, and core agent execution.

## Systems Involved

- [ACP Adapter](../entities/acp-adapter.md)
- [Agent Loop Runtime](../entities/agent-loop-runtime.md)
- [Tool Registry and Dispatch](../entities/tool-registry-and-dispatch.md)
- [Config and Profile System](../entities/config-and-profile-system.md)

## Interaction Model

The bridge path is:

1. An editor starts `hermes acp` or `hermes-acp`.
2. `acp_adapter/entry.py` loads the Hermes environment and starts the stdio ACP server.
3. `HermesACPAgent` receives ACP lifecycle calls such as initialize, new session, prompt, cancel, or fork.
4. `SessionManager` stores live session state including cwd, history, model, and cancel event.
5. Prompt execution runs in a worker thread so the ACP event loop stays responsive.
6. Agent callbacks are converted into ACP session updates through the event bridge.
7. Approval-sensitive operations are rerouted through the ACP permission bridge.
8. Final tool and message outputs are rendered into editor-friendly ACP content blocks.

The bridge is therefore bidirectional: ACP commands create or control Hermes sessions, and Hermes runtime callbacks are translated back into ACP events.

## Key Interfaces

| Interface | Role |
|------|------|
| `HermesACPAgent` | Main ACP protocol implementation |
| `SessionManager` | Live session ownership and lookup |
| `ThreadPoolExecutor` in `acp_adapter/server.py` | Keeps synchronous `AIAgent` work off the async ACP loop |
| event bridge callbacks | Forward reasoning, tool progress, and messages back to the editor |
| permission bridge | Converts dangerous-command prompts into ACP permission requests |

The main boundary condition is process locality. ACP sessions are live objects inside the adapter process, not a distributed session store. Hermes still reuses broader runtime concepts such as cwd binding, model switching, toolset selection, and cancellation.

## Source Evidence

- `acp_adapter/entry.py` — boot path
- `acp_adapter/server.py` — ACP method handling, worker-thread execution, MCP registration refresh
- `acp_adapter/session.py` — session state store
- `acp_adapter/events.py` and `permissions.py` — event and approval bridging
- `website/docs/developer-guide/acp-internals.md` — maintainer explanation of the bridge model

## See Also

- [ACP Adapter](../entities/acp-adapter.md)
- [Interruption and Human Approval Flow](../concepts/interruption-and-human-approval-flow.md)
- [CLI Runtime](../entities/cli-runtime.md)
- [Agent Loop Runtime](../entities/agent-loop-runtime.md)
