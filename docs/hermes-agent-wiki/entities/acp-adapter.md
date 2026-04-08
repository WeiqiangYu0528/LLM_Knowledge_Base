# ACP Adapter

## Overview

The ACP adapter exposes Hermes to editors through the Agent Client Protocol. It wraps the synchronous `AIAgent` runtime in an async JSON-RPC stdio server, manages editor-scoped sessions, bridges runtime callbacks into ACP events, and translates Hermes approval prompts into ACP permission requests.

## Key Types / Key Concepts

| Type or function | Role |
|------|------|
| `HermesACPAgent` in `acp_adapter/server.py` | ACP protocol implementation wrapping Hermes |
| `SessionManager` in `acp_adapter/session.py` | Tracks live ACP sessions and their state |
| event bridge in `acp_adapter/events.py` | Converts agent callbacks into ACP updates |
| permission bridge in `acp_adapter/permissions.py` | Maps Hermes approval decisions into ACP permission flows |
| `hermes-acp` toolset | Restricted tool surface used for editor sessions |

## Architecture

The ACP subsystem is a transport bridge with four responsibilities:

1. boot a stdio JSON-RPC server
2. manage session objects that carry cwd, history, model, and cancel state
3. run synchronous agent work in background worker threads
4. surface progress, messages, and approvals back to the editor protocol

That means ACP owns session transport and editor UX translation, while still delegating core execution to `AIAgent`.

## Runtime Behavior

### Boot and session lifecycle

Boot starts from `acp_adapter/entry.py`, loads the Hermes environment, configures logging to stderr, constructs `HermesACPAgent`, and runs the ACP server. Session methods such as `new`, `load`, `resume`, and `fork` all work through `SessionManager`, which stores the live agent instance plus cwd, history, model, and cancel event.

### Worker-thread execution

ACP I/O lives on an async event loop, but `AIAgent` is synchronous. The adapter resolves that by running prompt execution in a `ThreadPoolExecutor`. The event bridge then uses `asyncio.run_coroutine_threadsafe(...)` to forward updates safely back onto the ACP loop.

### Permission and tool rendering

Dangerous terminal approvals cannot reuse CLI prompts or gateway messages. The ACP permission bridge converts them into ACP permission requests. Tool rendering helpers also translate Hermes tool results into editor-friendly views such as diffs, terminal blocks, or truncated previews.

## Source Files

| File | Purpose |
|------|---------|
| `acp_adapter/entry.py` | Boot flow and stdio server startup |
| `acp_adapter/server.py` | `HermesACPAgent`, session methods, MCP registration, worker-thread execution |
| `acp_adapter/session.py` | Live session tracking and fork/cancel support |
| `acp_adapter/events.py` | Callback-to-ACP event bridge |
| `acp_adapter/permissions.py` | Approval translation layer |
| `website/docs/developer-guide/acp-internals.md` | Maintainer guide to lifecycle and bridge design |

## See Also

- [CLI Runtime](cli-runtime.md)
- [Gateway Runtime](gateway-runtime.md)
- [ACP Editor Session Bridge](../syntheses/acp-editor-session-bridge.md)
- [Interruption and Human Approval Flow](../concepts/interruption-and-human-approval-flow.md)
