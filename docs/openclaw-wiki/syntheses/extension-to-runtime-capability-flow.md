# Extension to Runtime Capability Flow

## Overview

This synthesis explains how a bundled or installed extension becomes an active runtime capability in OpenClaw.

This synthesis is where the control-plane nature of OpenClaw becomes visible, because no single entity page can show how routing, sessions, plugins, devices, and channels cooperate. The point of this page is to make the handoff between systems explicit so a reader can reason about failures, extension points, and policy boundaries without re-deriving the whole path from raw source.

## Systems Involved

| System | Contribution |
| --- | --- |
| [Pluginized Capability Delivery](../concepts/pluginized-capability-delivery.md) | Providers, channels, and tools delivered through plugin activation |
| [Gateway as Control Plane](../concepts/gateway-as-control-plane.md) | Why the gateway owns sessions, channels, tools, config, and UI surfaces |
| [Plugin Platform](../entities/plugin-platform.md) | Loader, manifests, install/uninstall, registry state, and runtime activation |
| [Provider and Model System](../entities/provider-and-model-system.md) | Provider catalogs, auth choices, model selection, and failover surfaces |
| [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md) | Related page in this wiki. |
| [Architecture Overview](../summaries/architecture-overview.md) | High-level orientation to OpenClaw as a local-first personal assistant platform |

## Interaction Model

Read this interaction model as a boundary-by-boundary path rather than a single function call:
1. An extension is discovered from bundled or installable plugin roots.
2. Its manifest is loaded and validated.
3. Activation policy determines whether the plugin should be enabled.
4. Runtime APIs and registries are updated with the plugin's capabilities.
5. Gateway, channels, providers, or agents begin consuming the new capability.
6. [Plugin Platform](../entities/plugin-platform.md) contributes one stage of the cross-system path described by this synthesis.
7. [Provider and Model System](../entities/provider-and-model-system.md) contributes one stage of the cross-system path described by this synthesis.
8. [Channel Plugin Adapters](../entities/channel-plugin-adapters.md) contributes one stage of the cross-system path described by this synthesis.
9. [Gateway Control Plane](../entities/gateway-control-plane.md) contributes one stage of the cross-system path described by this synthesis.

What matters here is where ownership changes hands. One subsystem usually receives input, another owns durable state, another chooses policy or execution strategy, and another surfaces the result back to a user or peer system. If those boundaries are not visible, the runtime looks simpler than it actually is and changes become riskier.

## Key Interfaces

| Interface Or Contract | Why It Matters |
| --- | --- |
| `Central discovery, load, validation, and activation path` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Active registry lifecycle and runtime state` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Provider inventory and catalog behavior` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Runtime-facing provider assembly` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Main export surface for channel plugin integration` | Boundary surface called out by the current page or inferred from the cited sources. |

## Operational Consequences

The synthesis view is usually where debugging stops being guesswork. A symptom that appears in the UI or CLI often originates several layers deeper, especially in repos that separate session state, routing, protocol bridging, plugin activation, and persistence. By following the sequence above, an implementer can decide whether a bug belongs in the initiating surface, the policy layer, the execution layer, or the state-reconciliation layer.

This also explains why change review should span more than one page. A modification that looks local inside an entity page may still alter a synthesis if it changes when state is persisted, which system chooses the active model, how approvals are routed, or where an outbound reply is emitted.

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `src/plugins/loader.ts` | Central discovery, load, validation, and activation path |
| `src/plugins/runtime.ts` | Active registry lifecycle and runtime state |
| `src/plugins/provider-catalog.ts` | Provider inventory and catalog behavior |
| `src/plugins/providers.runtime.ts` | Runtime-facing provider assembly |
| `src/channels/plugins/index.ts` | Main export surface for channel plugin integration |
| `src/channels/plugins/load.ts` | Channel plugin load path |

## See Also

- [Pluginized Capability Delivery](../concepts/pluginized-capability-delivery.md)
- [Gateway as Control Plane](../concepts/gateway-as-control-plane.md)
- [Plugin Platform](../entities/plugin-platform.md)
- [Provider and Model System](../entities/provider-and-model-system.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
