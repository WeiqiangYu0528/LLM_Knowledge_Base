# Cross-Session Recall and Memory Provider Pluggability

## Overview

Hermes combines two recall mechanisms that serve different purposes: search over concrete past sessions, and pluggable providers that can maintain their own higher-level memory representation. The system is designed so the agent can use both without conflating them.

## Mechanism

The base recall path is session storage plus `session_search`:

1. SQLite stores the full conversation record.
2. FTS5 finds relevant messages across sessions.
3. Hermes loads the best matching sessions.
4. An auxiliary model summarizes them into focused recall blocks.

External memory providers then add a second path:

1. the provider contributes static system-prompt information
2. the provider can prefetch turn-relevant context
3. the provider can sync each completed turn to an external backend
4. the provider may expose additional tools

The built-in provider is always on. Exactly one external provider may be active.

## Involved Entities

- [Memory and Learning Loop](../entities/memory-and-learning-loop.md)
- [Session Storage](../entities/session-storage.md)
- [Plugin and Memory Provider System](../entities/plugin-and-memory-provider-system.md)
- [Prompt Assembly System](../entities/prompt-assembly-system.md)

## Source Evidence

- `tools/session_search_tool.py` — FTS5 search, session loading, auxiliary summarization
- `agent/memory_manager.py` — provider orchestration and recall injection
- `agent/memory_provider.py` — provider lifecycle contract
- `plugins/memory/__init__.py` — provider discovery and one-provider activation model

## See Also

- [Memory and Learning Loop](../entities/memory-and-learning-loop.md)
- [Session Storage](../entities/session-storage.md)
- [Self-Improving Agent Architecture](self-improving-agent-architecture.md)
- [Compression Memory and Session Search Loop](../syntheses/compression-memory-and-session-search-loop.md)
