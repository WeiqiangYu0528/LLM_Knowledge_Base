# Hermes Agent Glossary

> Stable vocabulary used across the Hermes Agent runtime, docs, and wiki pages.

## Overview

Hermes mixes product-facing language, filesystem conventions, and runtime-specific terms. This glossary normalizes the most important ones so the rest of the wiki can stay implementation-led without re-explaining every term from scratch.

## Terms

| Term | Meaning in Hermes | Primary page |
|------|-------------------|--------------|
| `HERMES_HOME` | The profile-scoped home directory containing `config.yaml`, `.env`, `SOUL.md`, logs, skills, memories, cron state, and session data | [Config and Profile System](../entities/config-and-profile-system.md) |
| `SOUL.md` | The agent identity file loaded as the first prompt layer when present | [Prompt Assembly System](../entities/prompt-assembly-system.md) |
| `toolset` | A named bundle of tools exposed to a session, often tied to a product surface such as `hermes-cli` or `hermes-acp` | [Tool Registry and Dispatch](../entities/tool-registry-and-dispatch.md) |
| `api_mode` | The transport shape used for model calls: `chat_completions`, `codex_responses`, or `anthropic_messages` | [Provider Runtime](../entities/provider-runtime.md) |
| `session lineage` | Parent-child relationship between sessions, typically created when context compression forks a new child session | [Session Storage](../entities/session-storage.md) |
| `gateway session key` | Deterministic routing key such as `agent:main:telegram:private:12345` used to isolate messaging conversations | [Gateway Runtime](../entities/gateway-runtime.md) |
| `pairing` | Gateway-side authorization flow that lets an existing authorized user approve a new messaging user | [Gateway Runtime](../entities/gateway-runtime.md) |
| `Honcho` | A specific external memory-provider ecosystem with its own tools, identity concepts, and migration path | [Memory and Learning Loop](../entities/memory-and-learning-loop.md) |
| `ACP` | Agent Client Protocol; Hermes uses it to expose editor-integrated sessions over JSON-RPC stdio | [ACP Adapter](../entities/acp-adapter.md) |
| `context compression` | Summarization and pruning of older turns when a conversation grows too large for the model context window | [Agent Loop Runtime](../entities/agent-loop-runtime.md) |
| `gateway hygiene compression` | Pre-agent compression pass in the gateway that keeps dormant long-lived messaging sessions from overrunning context | [Compression Memory and Session Search Loop](../syntheses/compression-memory-and-session-search-loop.md) |
| `prefetch` | Memory-provider recall step that injects background context before an API call | [Memory and Learning Loop](../entities/memory-and-learning-loop.md) |
| `session_search` | A tool and runtime path for querying past sessions from the SQLite store | [Memory and Learning Loop](../entities/memory-and-learning-loop.md) |
| `skill` | Instructional capability packaged as a directory with `SKILL.md` and optional helpers | [Skills System](../entities/skills-system.md) |
| `optional skill` | Official but not automatically installed skill distributed under `optional-skills/` | [Skills System](../entities/skills-system.md) |
| `memory provider` | An implementation of `MemoryProvider` that augments built-in memory with an external backend | [Plugin and Memory Provider System](../entities/plugin-and-memory-provider-system.md) |
| `running-agent guard` | Gateway logic that queues or interrupts incoming messages when a session is already executing | [Gateway Message to Agent Reply Flow](../syntheses/gateway-message-to-agent-reply-flow.md) |
| `approval callback` | Surface-specific mechanism for handling dangerous terminal or tool actions that require user approval | [Interruption and Human Approval Flow](../concepts/interruption-and-human-approval-flow.md) |
| `ToolContext` | Research/eval helper that exposes the same sandbox state the model used during rollout | [Research and Batch Surfaces](../entities/research-and-batch-surfaces.md) |

## Operational Notes

Several terms are overloaded across the repo:

- `memory` can mean the built-in `MEMORY.md` file, the external memory-provider ecosystem, or the user-facing memory tool. The relevant meaning depends on whether the page is discussing prompt assembly, storage, or tool routing.
- `plugin` can mean the general CLI plugin system or the separate `plugins/memory/` provider family. The latter is repo-bundled and gated to one active external provider at a time.
- `session` can refer to a CLI conversation, a gateway thread, an ACP editor session, or a cron-run ephemeral session. The runtime keeps them related through shared storage rather than by making them literally the same process object.

## See Also

- [Architecture Overview](architecture-overview.md)
- [Config and Profile System](../entities/config-and-profile-system.md)
- [Skills System](../entities/skills-system.md)
- [Multi-Surface Session Continuity](../concepts/multi-surface-session-continuity.md)
