# Hermes Agent Codebase Map

> Map of the major Hermes Agent source surfaces and the wiki pages that explain them.

## Overview

The Hermes repository is large enough that directory names alone are not a useful mental model. Several critical runtime owners live as top-level files rather than packages, and the repo also includes its own user docs, developer docs, bundled skills, optional skills, tests, and research environments. This page maps those source areas onto the Hermes wiki so a reader can jump from a raw path to the page that explains its runtime role.

## Top-Level Runtime Surfaces

| Path | What it owns | Why it matters | Primary wiki page |
|------|--------------|----------------|-------------------|
| `run_agent.py` | `AIAgent`, core turn loop, callbacks, compression hooks, fallback logic | Runtime center of gravity for most surfaces | [Agent Loop Runtime](../entities/agent-loop-runtime.md) |
| `model_tools.py` | Tool discovery, tool schema assembly, dispatch integration | Bridges the agent loop to the registry and dynamic tool surfaces | [Tool Registry and Dispatch](../entities/tool-registry-and-dispatch.md) |
| `hermes_state.py` | SQLite schema, FTS5, session lineage, contention handling | Canonical session persistence layer | [Session Storage](../entities/session-storage.md) |
| `batch_runner.py` | Dataset-driven parallel rollouts and trajectory capture | Main batch/eval/data-generation shell | [Research and Batch Surfaces](../entities/research-and-batch-surfaces.md) |
| `toolsets.py` | Toolset definitions and presets | Governs which capabilities sessions expose | [Toolset-Based Capability Governance](../concepts/toolset-based-capability-governance.md) |

## Package-Level Ownership Map

| Path | Runtime role | Notes | Primary wiki page |
|------|--------------|-------|-------------------|
| `agent/` | Prompt assembly, compression, caching, model metadata, memory orchestration | Houses most of the runtime internals that `AIAgent` delegates to | [Prompt Assembly System](../entities/prompt-assembly-system.md), [Memory and Learning Loop](../entities/memory-and-learning-loop.md) |
| `hermes_cli/` | CLI entry points, setup wizard, config, auth, command registry, model switching | Shared configuration surface for several non-CLI shells too | [CLI Runtime](../entities/cli-runtime.md), [Config and Profile System](../entities/config-and-profile-system.md) |
| `tools/` | Self-registering tool modules, registry, approval logic, terminal and browser backends | The capability surface the model sees | [Tool Registry and Dispatch](../entities/tool-registry-and-dispatch.md), [Terminal and Execution Environments](../entities/terminal-and-execution-environments.md) |
| `gateway/` | Long-running messaging runtime, session context, delivery, pairing, hooks, platform adapters | Hermes as a messaging agent platform | [Gateway Runtime](../entities/gateway-runtime.md), [Messaging Platform Adapters](../entities/messaging-platform-adapters.md) |
| `cron/` | Scheduled jobs and scheduler loop | Isolated automation shell on top of the main runtime | [Cron System](../entities/cron-system.md) |
| `acp_adapter/` | ACP server, session manager, event and permission bridge | Editor-facing transport layer for Hermes | [ACP Adapter](../entities/acp-adapter.md) |
| `plugins/memory/` | Repo-bundled external memory-provider plugins | Separate plugin family from generic CLI plugins | [Plugin and Memory Provider System](../entities/plugin-and-memory-provider-system.md) |
| `environments/` | RL/eval environment framework, tool parsers, benchmark shells | Training and evaluation surfaces tied to Hermes tools | [Research and Batch Surfaces](../entities/research-and-batch-surfaces.md) |

## Documentation and Knowledge Sources

| Path | Why it matters | Primary wiki page |
|------|----------------|-------------------|
| `README.md` | Product-level framing, install flow, user-visible features | [Architecture Overview](architecture-overview.md) |
| `website/docs/developer-guide/architecture.md` | Repo-wide subsystem map maintained by the Hermes project itself | [Architecture Overview](architecture-overview.md) |
| `website/docs/developer-guide/agent-loop.md` | Best high-level guide to `AIAgent` internals | [Agent Loop Runtime](../entities/agent-loop-runtime.md) |
| `website/docs/developer-guide/prompt-assembly.md` | Explains cached vs ephemeral prompt construction | [Prompt Assembly System](../entities/prompt-assembly-system.md) |
| `website/docs/developer-guide/provider-runtime.md` | Provider, credential, and fallback behavior | [Provider Runtime](../entities/provider-runtime.md) |
| `website/docs/developer-guide/tools-runtime.md` | Registry, toolsets, dispatch, approval, environments | [Tool Registry and Dispatch](../entities/tool-registry-and-dispatch.md) |
| `website/docs/developer-guide/gateway-internals.md` | Gateway ownership boundaries and message flow | [Gateway Runtime](../entities/gateway-runtime.md) |
| `website/docs/developer-guide/session-storage.md` | SQLite schema and lineage model | [Session Storage](../entities/session-storage.md) |

The developer-guide pages matter because they reveal the subsystem boundaries the repo itself considers stable. They are not authoritative on every implementation detail, but they are excellent maps for deciding which code files to inspect next.

## Product and Extension Surfaces

Hermes has several surfaces that are easy to overlook if you only read the runtime spine:

- `skills/` and `optional-skills/` define a large instruction-layer capability catalog. They matter to runtime behavior because skill discovery and activation modify the prompt and sandbox environment.
- `gateway/platforms/` contains platform-specific adapters for Telegram, Discord, Slack, Signal, WhatsApp, Matrix, Home Assistant, and enterprise chat systems such as DingTalk, Feishu, and WeCom.
- `tests/` is not just unit coverage; it encodes assumptions around fallback models, session persistence, ACP bridging, and memory-plugin behavior.
- `website/docs/user-guide/` describes user-facing expectations that often correspond to hidden configuration or routing choices in code.

## Suggested Reading Paths

### To understand the agent core

Start with `run_agent.py`, then `agent/prompt_builder.py`, `hermes_cli/runtime_provider.py`, `model_tools.py`, and `hermes_state.py`. That path maps onto [Agent Loop Runtime](../entities/agent-loop-runtime.md), [Prompt Assembly System](../entities/prompt-assembly-system.md), [Provider Runtime](../entities/provider-runtime.md), and [Session Storage](../entities/session-storage.md).

### To understand messaging and long-running operation

Read `gateway/run.py`, `gateway/session.py`, `gateway/delivery.py`, `gateway/pairing.py`, and `gateway/platforms/base.py`, then move to [Gateway Runtime](../entities/gateway-runtime.md), [Messaging Platform Adapters](../entities/messaging-platform-adapters.md), and [Gateway Message to Agent Reply Flow](../syntheses/gateway-message-to-agent-reply-flow.md).

### To understand automation and editor integration

Read `cron/jobs.py`, `cron/scheduler.py`, `acp_adapter/server.py`, `acp_adapter/session.py`, and `acp_adapter/permissions.py`, then move to [Cron System](../entities/cron-system.md), [ACP Adapter](../entities/acp-adapter.md), and [ACP Editor Session Bridge](../syntheses/acp-editor-session-bridge.md).

## See Also

- [Architecture Overview](architecture-overview.md)
- [CLI Runtime](../entities/cli-runtime.md)
- [Gateway Runtime](../entities/gateway-runtime.md)
- [Research and Batch Surfaces](../entities/research-and-batch-surfaces.md)
