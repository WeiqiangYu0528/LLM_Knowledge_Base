# Compression Memory and Session Search Loop

## Overview

Long-running Hermes sessions remain usable because three subsystems cooperate instead of competing: context compression keeps the active window bounded, built-in or external memory preserves durable facts, and session search rehydrates relevant older work on demand. This synthesis explains how those mechanisms complement one another.

## Systems Involved

- [Agent Loop Runtime](../entities/agent-loop-runtime.md)
- [Prompt Assembly System](../entities/prompt-assembly-system.md)
- [Memory and Learning Loop](../entities/memory-and-learning-loop.md)
- [Session Storage](../entities/session-storage.md)

## Interaction Model

The feedback loop looks like this:

1. A conversation grows across many turns and tool calls.
2. Before the context window is exhausted, Hermes may prune or summarize older turns.
3. Before compaction, memory providers can flush or extract state so knowledge is not lost.
4. The compressed conversation continues as a child session with lineage preserved.
5. Later, when a user asks about related past work, the agent can invoke `session_search`.
6. `session_search` uses FTS5 over persisted sessions, loads promising conversations, and uses an auxiliary model to summarize them.
7. That summarized recall re-enters the current turn as context, while built-in or external memory providers may also prefetch relevant durable facts.

Compression therefore does not mean "older work disappears". It means older work moves into structured storage and summary channels that can be reintroduced when useful.

## Key Interfaces

| Interface | Role |
|------|------|
| `ContextCompressor` and compression hooks | Bound active context size while preserving recent critical turns |
| `MemoryManager.on_pre_compress(...)` | Gives providers a chance to extract knowledge before older turns are compacted |
| SQLite FTS5 tables | Support later search across old sessions |
| `session_search` tool | Rehydrates past work through summarization rather than dumping raw transcripts |

This composition works because each subsystem solves a different problem:

- compression protects the active window
- memory preserves durable facts
- session search recalls situationally relevant history

## Source Evidence

- `agent/context_compressor.py` and `run_agent.py` — compaction lifecycle
- `agent/memory_manager.py` and `agent/memory_provider.py` — pre-compress and sync hooks
- `hermes_state.py` — persisted messages and FTS5
- `tools/session_search_tool.py` — search plus auxiliary summarization
- `website/docs/developer-guide/context-compression-and-caching.md` — high-level explanation of the compression system

## See Also

- [Memory and Learning Loop](../entities/memory-and-learning-loop.md)
- [Session Storage](../entities/session-storage.md)
- [Cross-Session Recall and Memory Provider Pluggability](../concepts/cross-session-recall-and-memory-provider-pluggability.md)
- [Prompt Layering and Cache Stability](../concepts/prompt-layering-and-cache-stability.md)
