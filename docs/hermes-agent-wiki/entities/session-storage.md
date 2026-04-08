# Session Storage

## Overview

Hermes uses SQLite as its canonical session store. `hermes_state.py` owns the CLI-oriented database for message history, titles, usage accounting, and full-text search, while `gateway/session.py` adds gateway-specific session context and routing metadata on top. Together they give Hermes a durable conversation substrate that can survive process restarts, support search across sessions, and preserve lineage when long contexts are compressed.

## Key Types

| Type or table | Role |
|------|------|
| `SessionDB` in `hermes_state.py` | Main SQLite owner for session metadata and messages |
| `sessions` table | One row per session, including lineage, model, and billing fields |
| `messages` table | Full message history with tool-call and reasoning fields |
| `messages_fts` table | FTS5 index used for full-text search |
| `SessionStore` in `gateway/session.py` | Gateway-facing session context and routing store |

## Architecture

The persistence model has two related but distinct layers:

- `hermes_state.py` stores the canonical message history and metadata that multiple Hermes processes can share.
- `gateway/session.py` stores session-source context and message-routing details that are specific to messaging surfaces.

The shared SQLite store is the foundation for CLI history, session browsing, compression lineage, and session search. Gateway-specific session objects extend that with platform, chat, thread, and authorization context.

## Runtime Behavior

### SQLite schema and search

The main database lives at `~/.hermes/state.db` by default and uses WAL mode. The schema centers on:

- `sessions`
- `messages`
- `messages_fts`
- `schema_version`

The FTS5 table is synchronized via triggers, which makes full-text search a normal part of runtime behavior rather than an offline export feature.

### Lineage and compression

Hermes does not overwrite old sessions when compressing context. Instead it can create parent-child relationships through `parent_session_id`. That gives the system a stable way to say "this child session contains the continuing live conversation after older turns were compacted". It is a crucial detail for cross-session reasoning and for keeping compressed conversations traceable.

### Multi-process contention handling

The repo expects several writers to share one database: CLI sessions, gateway processes, worktree agents, and possibly background flows. `SessionDB` therefore uses:

- short SQLite timeouts
- application-level retry with jitter
- `BEGIN IMMEDIATE` to surface write contention early
- periodic WAL checkpoints

This is one of the pages where the implementation detail matters directly. Without these choices, the gateway and concurrent agents would spend too much time blocking or convoying on database locks.

### Gateway session context

`gateway/session.py` adds a `SessionSource` and `SessionContext` layer that tells Hermes where a message came from, how to describe it in the prompt, and how to route a reply back. `build_session_key(...)` and `SessionStore` are therefore part of the persistence story even though they are not stored in the main SQLite schema.

## Source Files

| File | Purpose |
|------|---------|
| `hermes_state.py` | SQLite schema, migrations, retries, FTS5, append/retrieve operations |
| `website/docs/developer-guide/session-storage.md` | Maintainer description of tables, triggers, and contention choices |
| `gateway/session.py` | Gateway session keys, source metadata, and prompt-context integration |
| `tests/` | Behavioral coverage around fallback, storage, and session browsing |

## See Also

- [Gateway Runtime](gateway-runtime.md)
- [Memory and Learning Loop](memory-and-learning-loop.md)
- [Multi-Surface Session Continuity](../concepts/multi-surface-session-continuity.md)
- [Compression Memory and Session Search Loop](../syntheses/compression-memory-and-session-search-loop.md)
