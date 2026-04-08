# Hermes Agent Architecture Wiki — Index

> Master catalog of the Hermes Agent sub-wiki.

| Meta | Value |
|------|-------|
| **Pages** | 34 |
| **Last Updated** | 2026-04-08 |
| **Last Lint** | 2026-04-08 |

---

## Summaries

| Page | Description |
|------|-------------|
| [Architecture Overview](summaries/architecture-overview.md) | Narrative orientation to Hermes Agent as a CLI, gateway, ACP, tool, memory, and automation platform |
| [Codebase Map](summaries/codebase-map.md) | Maps top-level packages, runtime owners, and documentation sources to Hermes wiki pages |
| [Glossary](summaries/glossary.md) | Stable Hermes vocabulary such as `SOUL.md`, toolsets, session lineage, ACP, and pairing |

## Entities

| Page | Description |
|------|-------------|
| [Agent Loop Runtime](entities/agent-loop-runtime.md) | `AIAgent` turn orchestration, retries, tool loops, budgets, callbacks, and fallback behavior |
| [Prompt Assembly System](entities/prompt-assembly-system.md) | Cached prompt construction, context-file priority, memory snapshots, and ephemeral overlays |
| [Provider Runtime](entities/provider-runtime.md) | Provider/model resolution, credentials, API modes, fallback, and auxiliary routing |
| [Tool Registry and Dispatch](entities/tool-registry-and-dispatch.md) | Tool registration, toolset filtering, dispatch, async bridging, and approval-sensitive execution |
| [Terminal and Execution Environments](entities/terminal-and-execution-environments.md) | Local and remote terminal backends plus sandbox-facing environment abstractions |
| [Skills System](entities/skills-system.md) | Skill discovery, frontmatter parsing, readiness gating, and CLI configuration for skills |
| [Memory and Learning Loop](entities/memory-and-learning-loop.md) | Built-in memory, external memory providers, session search, and skill-improvement surfaces |
| [Session Storage](entities/session-storage.md) | SQLite session persistence, lineage tracking, FTS5 search, and gateway session metadata |
| [CLI Runtime](entities/cli-runtime.md) | `hermes` command surface, startup sequence, setup flow, and slash-command infrastructure |
| [Config and Profile System](entities/config-and-profile-system.md) | `HERMES_HOME`, profiles, `config.yaml`, `.env`, defaults, and managed-install behavior |
| [Gateway Runtime](entities/gateway-runtime.md) | Long-running messaging runtime, authorization, session routing, delivery, and hooks |
| [Messaging Platform Adapters](entities/messaging-platform-adapters.md) | Normalized adapter contract across Telegram, Discord, Slack, Signal, and other platforms |
| [Cron System](entities/cron-system.md) | Scheduled jobs, job storage, gateway ticking, and isolated delivery behavior |
| [Plugin and Memory Provider System](entities/plugin-and-memory-provider-system.md) | General plugin loading plus the separate repo-bundled memory-provider plugin family |
| [ACP Adapter](entities/acp-adapter.md) | ACP server boot, session manager, event bridge, permission bridge, and editor-facing tool rendering |
| [Research and Batch Surfaces](entities/research-and-batch-surfaces.md) | Batch runner, RL/eval environments, tool-context reuse, and data-generation surfaces |

## Concepts

| Page | Description |
|------|-------------|
| [Self-Improving Agent Architecture](concepts/self-improving-agent-architecture.md) | Hermes treats memory, recall, skills, and session history as one improvement loop |
| [Prompt Layering and Cache Stability](concepts/prompt-layering-and-cache-stability.md) | Stable prompt prefixes are separated from per-turn overlays for cost and correctness reasons |
| [Toolset-Based Capability Governance](concepts/toolset-based-capability-governance.md) | Toolsets, readiness checks, and platform presets determine which actions the model can take |
| [Multi-Surface Session Continuity](concepts/multi-surface-session-continuity.md) | CLI, gateway, cron, and ACP all persist or reconstruct conversations through shared session machinery |
| [Environment Abstraction for Agent Execution](concepts/environment-abstraction-for-agent-execution.md) | Terminal and sandbox backends give Hermes one execution model across local and remote surfaces |
| [Cross-Session Recall and Memory Provider Pluggability](concepts/cross-session-recall-and-memory-provider-pluggability.md) | Built-in files, session DB search, and pluggable providers combine into Hermes recall behavior |
| [Interruption and Human Approval Flow](concepts/interruption-and-human-approval-flow.md) | Interrupt events, queued messages, and dangerous-command approvals preserve control during long runs |

## Syntheses

| Page | Description |
|------|-------------|
| [CLI to Agent Loop Composition](syntheses/cli-to-agent-loop-composition.md) | How the CLI resolves config, tools, providers, and session state before invoking `AIAgent` |
| [Gateway Message to Agent Reply Flow](syntheses/gateway-message-to-agent-reply-flow.md) | End-to-end path from platform ingress through authorization, session resolution, agent execution, and delivery |
| [Tool Call Execution and Approval Pipeline](syntheses/tool-call-execution-and-approval-pipeline.md) | How model tool calls become registry dispatches, environment actions, approvals, and tool results |
| [Compression Memory and Session Search Loop](syntheses/compression-memory-and-session-search-loop.md) | How compression, persistent memory, and session search keep long-lived conversations usable |
| [Cron Delivery and Platform Routing](syntheses/cron-delivery-and-platform-routing.md) | Scheduled jobs, fresh sessions, home-channel delivery, and routing isolation across messaging platforms |
| [ACP Editor Session Bridge](syntheses/acp-editor-session-bridge.md) | How ACP sessions wrap synchronous Hermes execution in an editor-safe async bridge |

## See Also

- [Hermes Agent Architecture Overview](summaries/architecture-overview.md)
- [Hermes Agent Wiki Schema](schema.md)
- [Hermes Agent Wiki Log](log.md)
- [Knowledge Base Home](../index.md)
