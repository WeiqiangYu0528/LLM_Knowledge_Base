# Memory and Learning Loop

## Overview

Hermes frames itself as a self-improving agent partly because memory is wired into the runtime rather than bolted on as a post-hoc search feature. The built-in memory files, session search, external memory-provider plugins, skill discovery, and skill-improvement surfaces all push the agent toward remembering durable facts and turning repeated workflows into reusable capabilities.

The concrete integration point is `MemoryManager` in `agent/memory_manager.py`. It orchestrates the always-on built-in provider plus at most one external provider, collects tool schemas, injects provider prompt blocks, handles recall prefetching, syncs completed turns, and lets special hooks run before session end or compression.

## Key Types / Key Concepts

| Type or concept | Role |
|------|------|
| `MemoryManager` in `agent/memory_manager.py` | Orchestrates built-in memory plus one external provider |
| `MemoryProvider` ABC in `agent/memory_provider.py` | Contract for external providers and provider tools |
| built-in memory provider | Always-on `MEMORY.md` / `USER.md` style persistence |
| session search | Runtime path for searching the SQLite session store across conversations |
| memory provider plugin | Optional external backend such as Honcho, Hindsight, Mem0, or OpenViking |

## Architecture

The memory subsystem intentionally combines several layers:

- built-in local memory files for durable facts
- SQLite-backed session persistence and search
- optional external memory providers in `plugins/memory/`
- prompt injection of recalled context
- tool surfaces that let the model explicitly write or query memory
- skill creation and improvement flows that turn repeated work into reusable instructions

These layers are related but not identical. Built-in memory is simple and always present. External providers offer richer recall or tooling but are limited to one active provider at a time so tool schemas and recall semantics do not explode.

## Runtime Behavior

### Provider registration and limits

`MemoryManager.add_provider()` always accepts the built-in provider first. It will only allow one non-builtin provider after that. This restriction is deliberate: multiple external providers would create overlapping tool surfaces and conflicting recall behavior. The manager also indexes provider-owned tool names so that tool calls can be routed back to the correct provider.

### Prompt contribution and recall

Memory affects the agent loop in two distinct ways:

- `build_system_prompt()` collects static provider prompt blocks.
- `prefetch_all()` collects dynamic recall context before an API call.

That split mirrors the broader prompt-assembly distinction between stable prompt layers and per-turn overlays. Static provider instructions belong in the stable prompt. Recalled context belongs near the active turn.

### Post-turn sync and session-end hooks

After each completed turn, the manager can:

- sync the turn to provider backends
- queue prefetch for the next turn
- mirror built-in memory writes
- run end-of-session extraction
- extract knowledge before compression discards older turns

These hooks are why memory participates in long-session survival rather than only passive storage.

### Learning beyond memory files

Hermes' "learning loop" also includes skills. The repo has large bundled and optional skill trees, and the runtime exposes skill discovery and viewing tools. Repeated workflows can therefore migrate from "agent remembers a fact" to "agent loads a reusable procedure". The memory subsystem and the skills subsystem are separate owners, but together they explain why Hermes talks about durable improvement rather than mere recall.

## Source Files

| File | Purpose |
|------|---------|
| `agent/memory_manager.py` | Orchestration of built-in plus external providers |
| `agent/memory_provider.py` | Abstract contract for provider lifecycle, recall, tools, and hooks |
| `plugins/memory/__init__.py` | Discovery and loading of repo-bundled external memory providers |
| `plugins/memory/*` | Provider implementations such as Honcho, Hindsight, Mem0, OpenViking, and others |
| `website/docs/developer-guide/memory-provider-plugin.md` | Maintainer guide for implementing provider plugins |

## See Also

- [Session Storage](session-storage.md)
- [Skills System](skills-system.md)
- [Cross-Session Recall and Memory Provider Pluggability](../concepts/cross-session-recall-and-memory-provider-pluggability.md)
- [Compression Memory and Session Search Loop](../syntheses/compression-memory-and-session-search-loop.md)
