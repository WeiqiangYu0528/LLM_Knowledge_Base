# Gateway Runtime

## Overview

The gateway is Hermes as a long-running messaging platform. `gateway/run.py` owns inbound event handling, session resolution, authorization, message guards, agent instantiation, slash-command dispatch, and response delivery. It is the shell that lets the same `AIAgent` runtime operate inside Telegram chats, Discord threads, Slack channels, Signal conversations, home-assistant hooks, and other messaging surfaces.

## Key Types

| Type or function | Role |
|------|------|
| `GatewayRunner` in `gateway/run.py` | Main long-running messaging runtime |
| `build_session_key(...)` in `gateway/session.py` | Deterministic routing key constructor |
| `SessionStore` in `gateway/session.py` | Gateway-facing storage and context owner |
| `gateway/delivery.py` | Outbound delivery routing and wrapping |
| `gateway/pairing.py` | Pairing-based authorization for new users |

## Architecture

The gateway runtime has four main responsibilities:

1. normalize external platform events into a shared internal event shape
2. decide whether a message belongs to an existing running session, a queued message, a slash command, or a new agent turn
3. instantiate or resume the runtime context needed for `AIAgent`
4. deliver the result back through the originating or configured destination platform

This is why the gateway owns its own `session.py`, `delivery.py`, `pairing.py`, `hooks.py`, and platform-adapter tree. It is not a trivial adapter around the CLI; it is a long-running control surface with routing and authorization rules of its own.

## Runtime Behavior

### Message ingress and session keys

Inbound platform adapters normalize raw events into a shared `MessageEvent` style object before calling into `GatewayRunner._handle_message()`. The gateway then constructs or reuses a session key, typically in a format such as `agent:main:{platform}:{chat_type}:{chat_id}`. Thread-aware platforms extend the key with thread identity when needed.

The result is that each conversation lane has a deterministic routing identity even when messages arrive over different transports or after restarts.

### Authorization and pairing

The gateway evaluates authorization in layers:

1. per-platform allow-all flags
2. platform allowlists
3. DM pairing flow
4. global allow-all
5. deny by default

This matters because the gateway is not assumed to be private. It needs a user-authorization model that can onboard new chat users without exposing the entire agent to arbitrary messages.

### Running-agent guard

The gateway has to deal with concurrent inbound messages while an agent is already working. It does that with a two-level guard:

- base adapter guard that queues messages and sets interrupt events early
- gateway-level guard that decides which commands bypass normal queuing

Commands such as `/approve`, `/deny`, `/stop`, or `/status` may need to reach the runner immediately even while the agent is blocked on an earlier turn.

### Delivery and hooks

Responses normally go back through the same adapter, but the gateway can also route background or cron results to configured home channels or explicit destinations. Hook infrastructure lets Hermes run lifecycle-specific integrations around startup, session start/end, agent steps, and commands.

## Source Files

| File | Purpose |
|------|---------|
| `gateway/run.py` | `GatewayRunner`, message handling, guard logic, command dispatch |
| `gateway/session.py` | Session keys, source metadata, session prompt context |
| `gateway/delivery.py` | Outbound routing, delivery wrapping, home-channel behavior |
| `gateway/pairing.py` | User pairing and authorization state |
| `website/docs/developer-guide/gateway-internals.md` | Maintainer description of routing, adapters, and guards |

## See Also

- [Messaging Platform Adapters](messaging-platform-adapters.md)
- [Session Storage](session-storage.md)
- [Interruption and Human Approval Flow](../concepts/interruption-and-human-approval-flow.md)
- [Gateway Message to Agent Reply Flow](../syntheses/gateway-message-to-agent-reply-flow.md)
