# Messaging Platform Adapters

## Overview

The gateway runtime relies on a normalized adapter contract so Hermes can speak to many messaging platforms without rewriting the higher-level routing logic each time. Each adapter handles transport-specific connection, authentication, media handling, and outbound delivery, but all of them feed normalized events into the gateway core.

## Key Types / Key Concepts

| Type or module | Role |
|------|------|
| `gateway/platforms/base.py` | Shared adapter interface and common media-cache helpers |
| `Platform` and platform config | Canonical identifiers for messaging surfaces |
| platform modules such as `telegram.py`, `discord.py`, `slack.py` | Transport-specific adapters |
| `SessionSource` | Normalized origin description injected into routing and prompts |

## Architecture

The adapter layer separates three concerns:

- platform-specific transport code
- normalized event and delivery contract
- gateway-owned session and authorization logic

This is why adapters are responsible for `connect()`, `disconnect()`, and `send_message()`, while the gateway runner is responsible for deciding whether an event should create a new turn, interrupt an active run, or dispatch a command.

`gateway/platforms/base.py` also owns reusable cache helpers for images and audio, plus a messaging-safe stance toward secret capture. Those are cross-platform concerns that do not belong in a single adapter.

## Runtime Behavior

### Normalization

Adapters turn raw platform payloads into normalized session source information, usually including:

- platform name
- chat ID
- chat type
- user identity
- optional thread identity

This normalized shape then feeds `build_session_key(...)` and the gateway prompt context.

### Media and platform-specific quirks

The base adapter layer includes local caching for downloaded images and audio because some platform URLs are temporary. Hermes wants the vision or STT path to operate on stable local files rather than platform-specific expiring URLs.

### Token and connection ownership

Some adapters use scoped locks so that two profiles do not accidentally share the same bot token simultaneously. This is one of the places where the profile system and the platform layer interact directly.

## Source Files

| File | Purpose |
|------|---------|
| `gateway/platforms/base.py` | Common interface, media caching, shared adapter helpers |
| `gateway/platforms/telegram.py`, `discord.py`, `slack.py`, `signal.py`, `whatsapp.py`, `matrix.py` | Major transport-specific adapters |
| `gateway/platforms/api_server.py`, `webhook.py`, `homeassistant.py` | Non-chat or integration-style adapters |
| `gateway/platforms/ADDING_A_PLATFORM.md` | Contributor guidance for adding new adapters |

## See Also

- [Gateway Runtime](gateway-runtime.md)
- [Session Storage](session-storage.md)
- [Gateway Message to Agent Reply Flow](../syntheses/gateway-message-to-agent-reply-flow.md)
- [Multi-Surface Session Continuity](../concepts/multi-surface-session-continuity.md)
