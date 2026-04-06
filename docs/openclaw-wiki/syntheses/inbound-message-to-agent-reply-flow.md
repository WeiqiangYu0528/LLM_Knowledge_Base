# Inbound Message to Agent Reply Flow

## Overview

This synthesis describes the main operational path for OpenClaw: a message arrives through a channel integration, gets routed into the correct session and agent, executes against the assistant runtime, and is delivered back through the correct channel surface.

This synthesis is where the control-plane nature of OpenClaw becomes visible, because no single entity page can show how routing, sessions, plugins, devices, and channels cooperate. The point of this page is to make the handoff between systems explicit so a reader can reason about failures, extension points, and policy boundaries without re-deriving the whole path from raw source.

## Systems Involved

| System | Contribution |
| --- | --- |
| [Multi Channel Session Routing](../concepts/multi-channel-session-routing.md) | How inbound channels and accounts map into isolated sessions and agents |
| [Gateway as Control Plane](../concepts/gateway-as-control-plane.md) | Why the gateway owns sessions, channels, tools, config, and UI surfaces |
| [Routing System](../entities/routing-system.md) | Account, peer, and binding-based route resolution into agent/session targets |
| [Channel Binding and Session Identity Flow](channel-binding-and-session-identity-flow.md) | How bindings, account IDs, routes, and session keys compose |
| [Architecture Overview](../summaries/architecture-overview.md) | High-level orientation to OpenClaw as a local-first personal assistant platform |
| [Codebase Map](../summaries/codebase-map.md) | Maps `src/`, `extensions/`, `apps/`, `skills/`, and `docs/` to the corresponding wiki pages |

## Interaction Model

Read this interaction model as a boundary-by-boundary path rather than a single function call:
1. A channel adapter receives an inbound event and normalizes channel/account metadata.
2. The routing system resolves the owning `agentId` and `sessionKey`.
3. The session system supplies transcript and lifecycle context.
4. The gateway invokes the relevant agent runtime path.
5. The resulting assistant output flows into reply dispatch and back out through the channel adapter.
6. [Gateway Control Plane](../entities/gateway-control-plane.md) contributes one stage of the cross-system path described by this synthesis.
7. [Channel System](../entities/channel-system.md) contributes one stage of the cross-system path described by this synthesis.
8. [Channel Plugin Adapters](../entities/channel-plugin-adapters.md) contributes one stage of the cross-system path described by this synthesis.
9. [Routing System](../entities/routing-system.md) contributes one stage of the cross-system path described by this synthesis.

What matters here is where ownership changes hands. One subsystem usually receives input, another owns durable state, another chooses policy or execution strategy, and another surfaces the result back to a user or peer system. If those boundaries are not visible, the runtime looks simpler than it actually is and changes become riskier.

## Key Interfaces

| Interface Or Contract | Why It Matters |
| --- | --- |
| `Main composition root for gateway startup and subsystem wiring` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Core control-plane method handlers` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Lightweight normalized channel lookup against active plugins` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Shared channel configuration model` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Main export surface for channel plugin integration` | Boundary surface called out by the current page or inferred from the cited sources. |

## Operational Consequences

The synthesis view is usually where debugging stops being guesswork. A symptom that appears in the UI or CLI often originates several layers deeper, especially in repos that separate session state, routing, protocol bridging, plugin activation, and persistence. By following the sequence above, an implementer can decide whether a bug belongs in the initiating surface, the policy layer, the execution layer, or the state-reconciliation layer.

This also explains why change review should span more than one page. A modification that looks local inside an entity page may still alter a synthesis if it changes when state is persisted, which system chooses the active model, how approvals are routed, or where an outbound reply is emitted.

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `src/gateway/server.impl.ts` | Main composition root for gateway startup and subsystem wiring |
| `src/gateway/server-methods.ts` | Core control-plane method handlers |
| `src/channels/registry.ts` | Lightweight normalized channel lookup against active plugins |
| `src/channels/channel-config.ts` | Shared channel configuration model |
| `src/channels/plugins/index.ts` | Main export surface for channel plugin integration |
| `src/channels/plugins/load.ts` | Channel plugin load path |

## See Also

- [Multi Channel Session Routing](../concepts/multi-channel-session-routing.md)
- [Gateway as Control Plane](../concepts/gateway-as-control-plane.md)
- [Routing System](../entities/routing-system.md)
- [Channel Binding and Session Identity Flow](channel-binding-and-session-identity-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
