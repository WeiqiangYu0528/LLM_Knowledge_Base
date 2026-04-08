# Gateway Message to Agent Reply Flow

## Overview

This page traces the end-to-end flow from an incoming platform message to the final reply Hermes sends back. It is the main synthesis for understanding why the gateway is more than a transport shim: several ownership boundaries change hands between platform adapters, gateway routing, session storage, approval control, and the agent loop.

![Hermes gateway message flow](../assets/graphs/hermes-gateway-message-flow.png)

[Edit diagram source](../assets/graphs/hermes-gateway-message-flow.excalidraw)

## Systems Involved

- [Messaging Platform Adapters](../entities/messaging-platform-adapters.md)
- [Gateway Runtime](../entities/gateway-runtime.md)
- [Session Storage](../entities/session-storage.md)
- [Provider Runtime](../entities/provider-runtime.md)
- [Agent Loop Runtime](../entities/agent-loop-runtime.md)
- [Tool Registry and Dispatch](../entities/tool-registry-and-dispatch.md)

## Interaction Model

The gateway path works in ordered stages:

1. A platform adapter receives a raw event from Telegram, Discord, Slack, Signal, or another supported surface.
2. The adapter normalizes it into Hermes' internal event/session-source shape.
3. The gateway's running-agent guard checks whether this session is already busy.
4. The gateway computes or reuses a deterministic session key using `build_session_key(...)`.
5. Authorization is evaluated through allowlists, allow-all flags, or pairing state.
6. The gateway decides whether the input is:
   - a slash command
   - a bypass command such as approval or stop
   - a queued message for a running session
   - a normal user turn
7. For a normal user turn, the gateway restores session history and constructs an `AIAgent` with gateway callbacks and session context.
8. `AIAgent.run_conversation()` executes, potentially making tool calls or triggering approvals.
9. The gateway wraps or routes the final response and sends it back through the originating adapter or a configured delivery target.

The critical ownership changes are:

- adapter -> gateway when raw transport data becomes a normalized event
- gateway -> agent loop when a turn becomes an `AIAgent` execution
- agent loop -> tool runtime when tool calls are emitted
- tool runtime -> gateway or ACP/CLI approval transport when human approval is required
- gateway -> adapter again when the final reply is delivered

## Key Interfaces

| Interface | Role in the handoff |
|------|------|
| adapter `on_message()` | Converts platform-specific payloads into Hermes events |
| `build_session_key(...)` | Gives routing and persistence a shared identity for the conversation lane |
| `_handle_message()` in `gateway/run.py` | Central gateway dispatcher |
| `AIAgent.run_conversation()` | Begins the core turn loop |
| `gateway/delivery.py` | Final outbound delivery and wrapping behavior |

The "running-agent guard" is especially important. It ensures that an already busy session does not accept arbitrary extra turns while still allowing urgent commands such as `/approve` or `/stop` to get through.

## Source Evidence

- `gateway/platforms/base.py` and concrete adapter modules — inbound normalization
- `gateway/run.py` — guard logic, authorization, slash-command routing, agent creation
- `gateway/session.py` — session key construction and source context
- `gateway/delivery.py` — outbound routing
- `website/docs/developer-guide/gateway-internals.md` — maintainer map of the same flow

## See Also

- [Gateway Runtime](../entities/gateway-runtime.md)
- [Messaging Platform Adapters](../entities/messaging-platform-adapters.md)
- [Interruption and Human Approval Flow](../concepts/interruption-and-human-approval-flow.md)
- [Tool Call Execution and Approval Pipeline](tool-call-execution-and-approval-pipeline.md)
