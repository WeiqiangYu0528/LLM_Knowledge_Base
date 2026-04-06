# Provider Tool Plugin Interaction Model

## Overview

This synthesis describes the three major policy surfaces that shape what an OpenCode agent can actually do in a given session: providers, tools, and plugins.

This synthesis matters because OpenCode is multi-surface; the runtime only makes sense when the handoff between client, session, provider, tool, and persistence layers is explicit. The point of this page is to make the handoff between systems explicit so a reader can reason about failures, extension points, and policy boundaries without re-deriving the whole path from raw source.

## Systems Involved

| System | Contribution |
| --- | --- |
| [Provider Agnostic Model Routing](../concepts/provider-agnostic-model-routing.md) | Uniform model selection across many provider SDKs |
| [Plugin Driven Extensibility](../concepts/plugin-driven-extensibility.md) | Hook-based extension through internal and external plugins |
| [Tool System](../entities/tool-system.md) | Tool registry, built-in tool catalog, plugin tools, and model-sensitive tool selection |
| [Permission System](../entities/permission-system.md) | Ask/allow/deny rules, permission requests, and approval lifecycle |
| [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md) | OpenCode core service with multiple clients layered on top |
| [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md) | Related page in this wiki. |

## Interaction Model

Read this interaction model as a boundary-by-boundary path rather than a single function call:
1. [Provider System](../entities/provider-system.md) contributes one stage of the cross-system path described by this synthesis.
2. [Tool System](../entities/tool-system.md) contributes one stage of the cross-system path described by this synthesis.
3. [Plugin System](../entities/plugin-system.md) contributes one stage of the cross-system path described by this synthesis.
4. [Permission System](../entities/permission-system.md) contributes one stage of the cross-system path described by this synthesis.

What matters here is where ownership changes hands. One subsystem usually receives input, another owns durable state, another chooses policy or execution strategy, and another surfaces the result back to a user or peer system. If those boundaries are not visible, the runtime looks simpler than it actually is and changes become riskier.

## Key Interfaces

| Interface Or Contract | Why It Matters |
| --- | --- |
| `Provider and model construction logic` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Provider/model identifiers and schemas` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Dynamic tool assembly and filtering` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Built-in tool implementations` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Plugin runtime, internal plugins, hook triggering, and external loading` | Boundary surface called out by the current page or inferred from the cited sources. |

## Operational Consequences

The synthesis view is usually where debugging stops being guesswork. A symptom that appears in the UI or CLI often originates several layers deeper, especially in repos that separate session state, routing, protocol bridging, plugin activation, and persistence. By following the sequence above, an implementer can decide whether a bug belongs in the initiating surface, the policy layer, the execution layer, or the state-reconciliation layer.

This also explains why change review should span more than one page. A modification that looks local inside an entity page may still alter a synthesis if it changes when state is persisted, which system chooses the active model, how approvals are routed, or where an outbound reply is emitted.

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `packages/opencode/src/provider/provider.ts` | Provider and model construction logic |
| `packages/opencode/src/provider/schema.ts` | Provider/model identifiers and schemas |
| `packages/opencode/src/tool/registry.ts` | Dynamic tool assembly and filtering |
| `packages/opencode/src/tool/registry.ts, tool.ts, bash.ts, read.ts, write.ts, task.ts` | Built-in tool implementations |
| `packages/opencode/src/plugin/index.ts` | Plugin runtime, internal plugins, hook triggering, and external loading |
| `packages/opencode/src/plugin/loader.ts` | Plugin loading and resolution |

## See Also

- [Provider Agnostic Model Routing](../concepts/provider-agnostic-model-routing.md)
- [Plugin Driven Extensibility](../concepts/plugin-driven-extensibility.md)
- [Tool System](../entities/tool-system.md)
- [Permission System](../entities/permission-system.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
