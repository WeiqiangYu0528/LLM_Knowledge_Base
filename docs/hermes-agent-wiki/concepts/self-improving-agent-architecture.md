# Self-Improving Agent Architecture

## Overview

Hermes calls itself a self-improving agent because the runtime is built to preserve useful facts, search past work, and turn repeated procedures into reusable assets instead of treating every turn as a fresh stateless prompt. This is not one feature. It is the composition of memory files, provider-backed recall, session search, compression summaries, and the large skills surface.

## Mechanism

The mechanism works in layers:

1. The prompt system injects stable identity, memory snapshots, and skill indexes at session start.
2. Before each turn, the memory manager can prefetch recall context from built-in or external providers.
3. During execution, the model can explicitly use `memory`, `session_search`, or skill-related tools to deepen recall.
4. After each turn, Hermes syncs completed conversation state back into memory providers or local stores.
5. When sessions become long, compression summarizes older turns instead of discarding them blindly, preserving decisions and next-step context.
6. Repeated workflows can graduate from "remembered fact" to a skill installed in the skill catalog.

The result is a feedback loop: prior sessions inform current execution, current execution writes back durable state, and durable state makes future sessions more capable.

## Involved Entities

- [Memory and Learning Loop](../entities/memory-and-learning-loop.md)
- [Prompt Assembly System](../entities/prompt-assembly-system.md)
- [Session Storage](../entities/session-storage.md)
- [Skills System](../entities/skills-system.md)
- [Agent Loop Runtime](../entities/agent-loop-runtime.md)

## Source Evidence

- `agent/memory_manager.py` — static prompt blocks, prefetch, sync, and provider hook orchestration
- `agent/memory_provider.py` — lifecycle contract for external providers
- `tools/session_search_tool.py` — cross-session recall via SQLite plus auxiliary summarization
- `tools/skills_tool.py` and `hermes_cli/skills_config.py` — skill discovery and activation surfaces
- `agent/context_compressor.py` and `run_agent.py` — retention of older session knowledge through compaction

## See Also

- [Cross-Session Recall and Memory Provider Pluggability](cross-session-recall-and-memory-provider-pluggability.md)
- [Compression Memory and Session Search Loop](../syntheses/compression-memory-and-session-search-loop.md)
- [Skills System](../entities/skills-system.md)
- [Memory and Learning Loop](../entities/memory-and-learning-loop.md)
