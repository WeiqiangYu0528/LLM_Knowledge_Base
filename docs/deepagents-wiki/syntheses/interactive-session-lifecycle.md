# Interactive Session Lifecycle

## Overview

This synthesis follows a user interaction through the interactive CLI, from startup to persistent thread state and back again on resume.

The value of this synthesis is in showing how Deep Agents composes opinionated defaults with optional extension points instead of treating those as separate products. The point of this page is to make the handoff between systems explicit so a reader can reason about failures, extension points, and policy boundaries without re-deriving the whole path from raw source.

## Systems Involved

| System | Contribution |
| --- | --- |
| [Session Persistence](../entities/session-persistence.md) | Thread IDs, SQLite checkpoints, metadata caches, and resume flow |
| [CLI Textual UI](../entities/cli-textual-ui.md) | Textual application, widgets, theme handling, and message rendering |
| [Context Management and Summarization](../concepts/context-management-and-summarization.md) | Auto-compaction, offloading, and context-window control |
| [Human in the Loop Approval](../concepts/human-in-the-loop-approval.md) | Interrupt-on policies, shell allow-lists, and approval-sensitive operations |
| [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md) | The planning, filesystem, subagent, and prompt bundle that defines Deep Agents |
| [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md) | Related page in this wiki. |

## Interaction Model

Read this interaction model as a boundary-by-boundary path rather than a single function call:
1. [CLI Runtime](../entities/cli-runtime.md) contributes one stage of the cross-system path described by this synthesis.
2. [CLI Textual UI](../entities/cli-textual-ui.md) contributes one stage of the cross-system path described by this synthesis.
3. [Session Persistence](../entities/session-persistence.md) contributes one stage of the cross-system path described by this synthesis.
4. [MCP System](../entities/mcp-system.md) contributes one stage of the cross-system path described by this synthesis.

What matters here is where ownership changes hands. One subsystem usually receives input, another owns durable state, another chooses policy or execution strategy, and another surfaces the result back to a user or peer system. If those boundaries are not visible, the runtime looks simpler than it actually is and changes become riskier.

## Key Interfaces

| Interface Or Contract | Why It Matters |
| --- | --- |
| `Main CLI entry point and startup logic` | Boundary surface called out by the current page or inferred from the cited sources. |
| `CLI agent assembly and shell approval handling` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Main Textual application` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Chat message rendering widgets` | Boundary surface called out by the current page or inferred from the cited sources. |
| `SQLite thread metadata, caches, and helper formatting` | Boundary surface called out by the current page or inferred from the cited sources. |

## Operational Consequences

The synthesis view is usually where debugging stops being guesswork. A symptom that appears in the UI or CLI often originates several layers deeper, especially in repos that separate session state, routing, protocol bridging, plugin activation, and persistence. By following the sequence above, an implementer can decide whether a bug belongs in the initiating surface, the policy layer, the execution layer, or the state-reconciliation layer.

This also explains why change review should span more than one page. A modification that looks local inside an entity page may still alter a synthesis if it changes when state is persisted, which system chooses the active model, how approvals are routed, or where an outbound reply is emitted.

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `libs/cli/deepagents_cli/main.py` | Main CLI entry point and startup logic |
| `libs/cli/deepagents_cli/agent.py` | CLI agent assembly and shell approval handling |
| `libs/cli/deepagents_cli/app.py` | Main Textual application |
| `libs/cli/deepagents_cli/widgets/messages.py` | Chat message rendering widgets |
| `libs/cli/deepagents_cli/sessions.py` | SQLite thread metadata, caches, and helper formatting |
| `libs/deepagents/deepagents/middleware/summarization.py` | Writes offloaded conversation history that complements persisted state |

## See Also

- [Session Persistence](../entities/session-persistence.md)
- [CLI Textual UI](../entities/cli-textual-ui.md)
- [Context Management and Summarization](../concepts/context-management-and-summarization.md)
- [Human in the Loop Approval](../concepts/human-in-the-loop-approval.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
