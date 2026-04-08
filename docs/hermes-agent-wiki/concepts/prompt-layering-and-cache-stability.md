# Prompt Layering and Cache Stability

## Overview

Hermes separates stable prompt state from per-turn overlays so that identity, memory, and project context remain predictable while transient runtime information stays outside the cached prefix. This is one of the most important design patterns in the repo because it affects both correctness and cost.

## Mechanism

The prompt builder first assembles a stable prefix from `SOUL.md`, tool-use guidance, memory snapshots, the skills index, and selected project context files. That prefix changes only when one of those stable inputs changes or the agent explicitly rebuilds it.

Per-turn data then enters later:

- ephemeral system messages
- budget warnings
- gateway overlays
- current-turn recall context
- prefill content

Because these overlays remain outside the stable prefix, Anthropic prompt caching can treat the earlier layers as reusable. At the same time, Hermes avoids rewriting identity or memory snapshots on every transient event.

## Involved Entities

- [Prompt Assembly System](../entities/prompt-assembly-system.md)
- [Agent Loop Runtime](../entities/agent-loop-runtime.md)
- [Memory and Learning Loop](../entities/memory-and-learning-loop.md)
- [Provider Runtime](../entities/provider-runtime.md)

## Source Evidence

- `agent/prompt_builder.py` — stable layer order, context-file priority, `SOUL.md` loading
- `agent/prompt_caching.py` — cache marker behavior
- `website/docs/developer-guide/prompt-assembly.md` — explicit stable-vs-ephemeral model
- `run_agent.py` — turn-time injection of warnings and overlays

## See Also

- [Prompt Assembly System](../entities/prompt-assembly-system.md)
- [Provider Runtime](../entities/provider-runtime.md)
- [CLI to Agent Loop Composition](../syntheses/cli-to-agent-loop-composition.md)
- [Self-Improving Agent Architecture](self-improving-agent-architecture.md)
