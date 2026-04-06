# Context Compaction and Session Recovery

## Overview

This synthesis explains how OpenCode manages long-running conversational state without reducing the runtime to an unbounded message log.

This synthesis matters because OpenCode is multi-surface; the runtime only makes sense when the handoff between client, session, provider, tool, and persistence layers is explicit. The point of this page is to make the handoff between systems explicit so a reader can reason about failures, extension points, and policy boundaries without re-deriving the whole path from raw source.

## Systems Involved

| System | Contribution |
| --- | --- |
| [Session System](../entities/session-system.md) | Session records, prompt selection, messages, compaction, retries, and summaries |
| [Storage and Sync](../entities/storage-and-sync.md) | JSON migration, persistence, event sourcing, and synchronization |
| [Tool and Agent Composition](../concepts/tool-and-agent-composition.md) | Build/plan agents, task delegation, and tool orchestration |
| [Request to Session Execution Flow](request-to-session-execution-flow.md) | How CLI/server requests become session-level model and tool activity |
| [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md) | OpenCode core service with multiple clients layered on top |
| [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md) | Related page in this wiki. |

## Interaction Model

Read this interaction model as a boundary-by-boundary path rather than a single function call:
1. [Session System](../entities/session-system.md) contributes one stage of the cross-system path described by this synthesis.
2. [Storage and Sync](../entities/storage-and-sync.md) contributes one stage of the cross-system path described by this synthesis.
3. [Permission System](../entities/permission-system.md) contributes one stage of the cross-system path described by this synthesis.
4. [Provider System](../entities/provider-system.md) contributes one stage of the cross-system path described by this synthesis.

What matters here is where ownership changes hands. One subsystem usually receives input, another owns durable state, another chooses policy or execution strategy, and another surfaces the result back to a user or peer system. If those boundaries are not visible, the runtime looks simpler than it actually is and changes become riskier.

## Key Interfaces

| Interface Or Contract | Why It Matters |
| --- | --- |
| `Session records, events, plan-file path logic, and usage aggregation` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Prompt selection logic` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Storage interface, migration logic, and JSON persistence helpers` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Database access layer` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Permission rules, request lifecycle, and service implementation` | Boundary surface called out by the current page or inferred from the cited sources. |

## Operational Consequences

The synthesis view is usually where debugging stops being guesswork. A symptom that appears in the UI or CLI often originates several layers deeper, especially in repos that separate session state, routing, protocol bridging, plugin activation, and persistence. By following the sequence above, an implementer can decide whether a bug belongs in the initiating surface, the policy layer, the execution layer, or the state-reconciliation layer.

This also explains why change review should span more than one page. A modification that looks local inside an entity page may still alter a synthesis if it changes when state is persisted, which system chooses the active model, how approvals are routed, or where an outbound reply is emitted.

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `packages/opencode/src/session/index.ts` | Session records, events, plan-file path logic, and usage aggregation |
| `packages/opencode/src/session/prompt.ts` | Prompt selection logic |
| `packages/opencode/src/storage/storage.ts` | Storage interface, migration logic, and JSON persistence helpers |
| `packages/opencode/src/storage/db.ts` | Database access layer |
| `packages/opencode/src/permission/index.ts` | Permission rules, request lifecycle, and service implementation |
| `packages/opencode/src/permission/evaluate.ts` | Rule evaluation logic |

## See Also

- [Session System](../entities/session-system.md)
- [Storage and Sync](../entities/storage-and-sync.md)
- [Tool and Agent Composition](../concepts/tool-and-agent-composition.md)
- [Request to Session Execution Flow](request-to-session-execution-flow.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
