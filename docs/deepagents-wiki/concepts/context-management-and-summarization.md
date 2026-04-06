# Context Management and Summarization

## Overview

Deep Agents treats context-window pressure as a first-class systems problem. Instead of passively accumulating history, it can summarize older context and offload it to backend-managed files.

This concept is easier to understand when read as a runtime guarantee rather than a marketing phrase: it describes how Deep Agents keeps customization and execution coherent across SDK, CLI, and remote backends. Context Management and Summarization is therefore best read as a mechanism with operational consequences, not merely a label for related features.

## Why It Exists

This concept exists because the Deep Agents codebase repeatedly needs a stable way to coordinate behavior across multiple entities without turning those entities into one monolith.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

## Mechanism

A useful way to read this mechanism is as an ordered path through the participating subsystems:
1. `SummarizationMiddleware` monitors message volume using token- or fraction-based thresholds.
2. When the trigger is exceeded, it summarizes older messages, keeps only a recent window, and writes evicted history to `/conversation_history/{thread_id}.md`.
3. A companion tool middleware can expose compaction as an explicit `compact_conversation` tool.
4. The middleware also includes a lighter optimization: truncating oversized tool-call arguments before full compaction is necessary.

The steps above are the operational skeleton. The exact file names vary by subsystem, but the concept remains stable because each participating entity contributes one predictable part of the chain. That is why the same concept can show up in SDK code, CLI wiring, plugin activation, channel routing, or persistence without becoming ambiguous.

## Invariants and Operational Implications

The most important invariant is that the linked entities are allowed to change implementation detail without changing the high-level guarantee described here. When a change breaks that guarantee, the failure usually appears at subsystem boundaries first: a summary no longer compacts correctly, a session route stops being stable, a skill path is not loaded consistently, or a permission rule is evaluated in the wrong layer.

Operationally, this means debugging should follow the mechanism rather than a UI symptom. Start where the concept is introduced, then inspect the next boundary where data, policy, or control is handed off. The source evidence table below is organized to support exactly that style of investigation.

## Involved Entities

| Entity | Role In This Concept |
| --- | --- |
| [Human in the Loop Approval](human-in-the-loop-approval.md) | Interrupt-on policies, shell allow-lists, and approval-sensitive operations |
| [Session Persistence](../entities/session-persistence.md) | Thread IDs, SQLite checkpoints, metadata caches, and resume flow |
| [Interactive Session Lifecycle](../syntheses/interactive-session-lifecycle.md) | End-to-end flow from CLI startup to persisted thread resume |
| [Evaluation Feedback Loop](../syntheses/evaluation-feedback-loop.md) | How the eval suite turns desired behavior into measurable architecture signals |
| [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md) | The planning, filesystem, subagent, and prompt bundle that defines Deep Agents |
| [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md) | Related page in this wiki. |

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `libs/deepagents/deepagents/graph.py` | Main `create_deep_agent` assembly logic and default base prompt |
| `libs/deepagents/deepagents/__init__.py` | Public SDK export surface |
| `libs/cli/deepagents_cli/sessions.py` | SQLite thread metadata, caches, and helper formatting |
| `libs/deepagents/deepagents/middleware/summarization.py` | Writes offloaded conversation history that complements persisted state |
| `libs/cli/deepagents_cli/main.py` | Main CLI entry point and startup logic |
| `libs/cli/deepagents_cli/agent.py` | CLI agent assembly and shell approval handling |

## See Also

- [Human in the Loop Approval](human-in-the-loop-approval.md)
- [Session Persistence](../entities/session-persistence.md)
- [Interactive Session Lifecycle](../syntheses/interactive-session-lifecycle.md)
- [Evaluation Feedback Loop](../syntheses/evaluation-feedback-loop.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
