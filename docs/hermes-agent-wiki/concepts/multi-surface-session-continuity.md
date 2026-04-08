# Multi-Surface Session Continuity

## Overview

Hermes lets the same broad runtime model operate in terminals, messaging platforms, scheduled jobs, and editors. Session continuity does not mean every surface shares one live process. It means they all reconstruct or persist enough state through shared storage and consistent routing rules that the user experiences them as durable conversations.

## Mechanism

The mechanism differs by surface but shares common primitives:

1. CLI sessions persist to SQLite through `SessionDB`.
2. Gateway sessions derive deterministic session keys from platform and chat context.
3. ACP sessions keep process-local live state but still reuse Hermes session concepts such as history, cwd, and cancellation.
4. Compression can fork child sessions while preserving lineage.
5. Session search can look across older sessions regardless of which surface created them.

Cron is the deliberate exception. Cron jobs run in fresh sessions because their isolation is part of the feature contract. Even there, however, Hermes still persists those sessions and routes deliveries through shared platform machinery.

## Involved Entities

- [Session Storage](../entities/session-storage.md)
- [Gateway Runtime](../entities/gateway-runtime.md)
- [CLI Runtime](../entities/cli-runtime.md)
- [ACP Adapter](../entities/acp-adapter.md)
- [Cron System](../entities/cron-system.md)

## Source Evidence

- `hermes_state.py` — shared message store and session metadata
- `gateway/session.py` — session-key construction and source context
- `acp_adapter/session.py` — live ACP session management
- `tools/session_search_tool.py` — cross-session recall
- `website/docs/developer-guide/session-storage.md` and `gateway-internals.md` — explicit session and routing descriptions

## See Also

- [Session Storage](../entities/session-storage.md)
- [Gateway Message to Agent Reply Flow](../syntheses/gateway-message-to-agent-reply-flow.md)
- [ACP Editor Session Bridge](../syntheses/acp-editor-session-bridge.md)
- [Cross-Session Recall and Memory Provider Pluggability](cross-session-recall-and-memory-provider-pluggability.md)
