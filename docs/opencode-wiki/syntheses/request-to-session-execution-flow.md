# Request to Session Execution Flow

## Overview

This synthesis follows an OpenCode request from CLI or server entry through project bootstrapping, session logic, model selection, tool execution, and persistence.

This synthesis matters because OpenCode is multi-surface; the runtime only makes sense when the handoff between client, session, provider, tool, and persistence layers is explicit. The point of this page is to make the handoff between systems explicit so a reader can reason about failures, extension points, and policy boundaries without re-deriving the whole path from raw source.

## Systems Involved

| System | Contribution |
| --- | --- |
| [Session System](../entities/session-system.md) | Session records, prompt selection, messages, compaction, retries, and summaries |
| [Tool System](../entities/tool-system.md) | Tool registry, built-in tool catalog, plugin tools, and model-sensitive tool selection |
| [Provider System](../entities/provider-system.md) | Provider/model loading, auth integration, and AI SDK routing |
| [Context Compaction and Session Recovery](context-compaction-and-session-recovery.md) | Prompt variants, summaries, compaction, retry, and restore behavior |
| [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md) | OpenCode core service with multiple clients layered on top |
| [Architecture Overview](../summaries/architecture-overview.md) | High-level orientation to the OpenCode monorepo, package boundaries, and runtime model |

## Interaction Model

Read this interaction model as a boundary-by-boundary path rather than a single function call:
1. [CLI Runtime](../entities/cli-runtime.md) contributes one stage of the cross-system path described by this synthesis.
2. [Project and Instance System](../entities/project-and-instance-system.md) contributes one stage of the cross-system path described by this synthesis.
3. [Session System](../entities/session-system.md) contributes one stage of the cross-system path described by this synthesis.
4. [Tool System](../entities/tool-system.md) contributes one stage of the cross-system path described by this synthesis.

What matters here is where ownership changes hands. One subsystem usually receives input, another owns durable state, another chooses policy or execution strategy, and another surfaces the result back to a user or peer system. If those boundaries are not visible, the runtime looks simpler than it actually is and changes become riskier.

## Key Interfaces

| Interface Or Contract | Why It Matters |
| --- | --- |
| `Root command entry point and startup/migration flow` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Project-scoped bootstrap helper` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Project-scoped instance context, caching, reload, and disposal` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Bootstrap sequence for plugins, file watchers, VCS, and snapshots` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Session records, events, plan-file path logic, and usage aggregation` | Boundary surface called out by the current page or inferred from the cited sources. |

## Operational Consequences

The synthesis view is usually where debugging stops being guesswork. A symptom that appears in the UI or CLI often originates several layers deeper, especially in repos that separate session state, routing, protocol bridging, plugin activation, and persistence. By following the sequence above, an implementer can decide whether a bug belongs in the initiating surface, the policy layer, the execution layer, or the state-reconciliation layer.

This also explains why change review should span more than one page. A modification that looks local inside an entity page may still alter a synthesis if it changes when state is persisted, which system chooses the active model, how approvals are routed, or where an outbound reply is emitted.

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `packages/opencode/src/index.ts` | Root command entry point and startup/migration flow |
| `packages/opencode/src/cli/bootstrap.ts` | Project-scoped bootstrap helper |
| `packages/opencode/src/project/instance.ts` | Project-scoped instance context, caching, reload, and disposal |
| `packages/opencode/src/project/bootstrap.ts` | Bootstrap sequence for plugins, file watchers, VCS, and snapshots |
| `packages/opencode/src/session/index.ts` | Session records, events, plan-file path logic, and usage aggregation |
| `packages/opencode/src/session/prompt.ts` | Prompt selection logic |

## See Also

- [Session System](../entities/session-system.md)
- [Tool System](../entities/tool-system.md)
- [Provider System](../entities/provider-system.md)
- [Context Compaction and Session Recovery](context-compaction-and-session-recovery.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
