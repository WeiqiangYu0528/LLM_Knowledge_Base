# Channel Binding and Session Identity Flow

## Overview

This synthesis explains how channel-level routing bindings and session identity combine to keep OpenClaw conversations isolated across accounts, peers, groups, and agent scopes.

This synthesis is where the control-plane nature of OpenClaw becomes visible, because no single entity page can show how routing, sessions, plugins, devices, and channels cooperate. The point of this page is to make the handoff between systems explicit so a reader can reason about failures, extension points, and policy boundaries without re-deriving the whole path from raw source.

## Systems Involved

| System | Contribution |
| --- | --- |
| [Multi Channel Session Routing](../concepts/multi-channel-session-routing.md) | How inbound channels and accounts map into isolated sessions and agents |
| [Gateway as Control Plane](../concepts/gateway-as-control-plane.md) | Why the gateway owns sessions, channels, tools, config, and UI surfaces |
| [Routing System](../entities/routing-system.md) | Account, peer, and binding-based route resolution into agent/session targets |
| [Session System](../entities/session-system.md) | Session identity, lifecycle, provenance, send policy, and transcript events |
| [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md) | Related page in this wiki. |
| [Architecture Overview](../summaries/architecture-overview.md) | High-level orientation to OpenClaw as a local-first personal assistant platform |

## Interaction Model

Read this interaction model as a boundary-by-boundary path rather than a single function call:
1. A channel adapter resolves normalized channel/account/peer metadata.
2. Bindings are evaluated to choose the effective target agent.
3. Session key helpers derive the persistence/concurrency lane.
4. Session lifecycle and transcript systems consume that identity.
5. Future inbound and outbound messages reuse the same lane unless policy says otherwise.
6. [Channel System](../entities/channel-system.md) contributes one stage of the cross-system path described by this synthesis.
7. [Channel Plugin Adapters](../entities/channel-plugin-adapters.md) contributes one stage of the cross-system path described by this synthesis.
8. [Routing System](../entities/routing-system.md) contributes one stage of the cross-system path described by this synthesis.
9. [Session System](../entities/session-system.md) contributes one stage of the cross-system path described by this synthesis.

What matters here is where ownership changes hands. One subsystem usually receives input, another owns durable state, another chooses policy or execution strategy, and another surfaces the result back to a user or peer system. If those boundaries are not visible, the runtime looks simpler than it actually is and changes become riskier.

## Key Interfaces

| Interface Or Contract | Why It Matters |
| --- | --- |
| `Lightweight normalized channel lookup against active plugins` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Shared channel configuration model` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Main export surface for channel plugin integration` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Channel plugin load path` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Main route resolution logic` | Boundary surface called out by the current page or inferred from the cited sources. |

## Operational Consequences

The synthesis view is usually where debugging stops being guesswork. A symptom that appears in the UI or CLI often originates several layers deeper, especially in repos that separate session state, routing, protocol bridging, plugin activation, and persistence. By following the sequence above, an implementer can decide whether a bug belongs in the initiating surface, the policy layer, the execution layer, or the state-reconciliation layer.

This also explains why change review should span more than one page. A modification that looks local inside an entity page may still alter a synthesis if it changes when state is persisted, which system chooses the active model, how approvals are routed, or where an outbound reply is emitted.

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `src/channels/registry.ts` | Lightweight normalized channel lookup against active plugins |
| `src/channels/channel-config.ts` | Shared channel configuration model |
| `src/channels/plugins/index.ts` | Main export surface for channel plugin integration |
| `src/channels/plugins/load.ts` | Channel plugin load path |
| `src/routing/resolve-route.ts` | Main route resolution logic |
| `src/routing/session-key.ts` | Session key construction and normalization helpers |

## See Also

- [Multi Channel Session Routing](../concepts/multi-channel-session-routing.md)
- [Gateway as Control Plane](../concepts/gateway-as-control-plane.md)
- [Routing System](../entities/routing-system.md)
- [Session System](../entities/session-system.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
