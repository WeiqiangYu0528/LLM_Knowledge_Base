# Onboarding to Live Gateway Flow

## Overview

This synthesis explains how OpenClaw goes from an unconfigured install to a live, reachable local assistant.

This synthesis is where the control-plane nature of OpenClaw becomes visible, because no single entity page can show how routing, sessions, plugins, devices, and channels cooperate. The point of this page is to make the handoff between systems explicit so a reader can reason about failures, extension points, and policy boundaries without re-deriving the whole path from raw source.

## Systems Involved

| System | Contribution |
| --- | --- |
| [Local First Personal Assistant Architecture](../concepts/local-first-personal-assistant-architecture.md) | OpenClaw as a user-owned assistant centered on a local gateway |
| [Pluginized Capability Delivery](../concepts/pluginized-capability-delivery.md) | Providers, channels, and tools delivered through plugin activation |
| [Gateway Control Plane](../entities/gateway-control-plane.md) | Gateway server startup, auth, sessions, plugin bootstrap, and control-plane duties |
| [Extension to Runtime Capability Flow](extension-to-runtime-capability-flow.md) | How extensions become active providers, channels, or tools in the runtime |
| [Gateway As Control Plane](../concepts/gateway-as-control-plane.md) | Related page in this wiki. |
| [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md) | Related page in this wiki. |

## Interaction Model

Read this interaction model as a boundary-by-boundary path rather than a single function call:
1. The user runs onboarding or setup-oriented CLI commands.
2. Config, env, and runtime prerequisites are checked.
3. Provider/channel/skill decisions populate config and activation state.
4. The gateway starts, loads runtime config, bootstraps plugins, and exposes control surfaces.
5. The assistant becomes reachable through configured channels and clients.
6. [CLI and Onboarding](../entities/cli-and-onboarding.md) contributes one stage of the cross-system path described by this synthesis.
7. [Gateway Control Plane](../entities/gateway-control-plane.md) contributes one stage of the cross-system path described by this synthesis.
8. [Plugin Platform](../entities/plugin-platform.md) contributes one stage of the cross-system path described by this synthesis.
9. [Provider and Model System](../entities/provider-and-model-system.md) contributes one stage of the cross-system path described by this synthesis.

What matters here is where ownership changes hands. One subsystem usually receives input, another owns durable state, another chooses policy or execution strategy, and another surfaces the result back to a user or peer system. If those boundaries are not visible, the runtime looks simpler than it actually is and changes become riskier.

## Key Interfaces

| Interface Or Contract | Why It Matters |
| --- | --- |
| `Top-level CLI bootstrap, env normalization, and command routing` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Commander program construction` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Main composition root for gateway startup and subsystem wiring` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Core control-plane method handlers` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Central discovery, load, validation, and activation path` | Boundary surface called out by the current page or inferred from the cited sources. |

## Operational Consequences

The synthesis view is usually where debugging stops being guesswork. A symptom that appears in the UI or CLI often originates several layers deeper, especially in repos that separate session state, routing, protocol bridging, plugin activation, and persistence. By following the sequence above, an implementer can decide whether a bug belongs in the initiating surface, the policy layer, the execution layer, or the state-reconciliation layer.

This also explains why change review should span more than one page. A modification that looks local inside an entity page may still alter a synthesis if it changes when state is persisted, which system chooses the active model, how approvals are routed, or where an outbound reply is emitted.

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `src/cli/run-main.ts` | Top-level CLI bootstrap, env normalization, and command routing |
| `src/cli/program.ts` | Commander program construction |
| `src/gateway/server.impl.ts` | Main composition root for gateway startup and subsystem wiring |
| `src/gateway/server-methods.ts` | Core control-plane method handlers |
| `src/plugins/loader.ts` | Central discovery, load, validation, and activation path |
| `src/plugins/runtime.ts` | Active registry lifecycle and runtime state |

## See Also

- [Local First Personal Assistant Architecture](../concepts/local-first-personal-assistant-architecture.md)
- [Pluginized Capability Delivery](../concepts/pluginized-capability-delivery.md)
- [Gateway Control Plane](../entities/gateway-control-plane.md)
- [Extension to Runtime Capability Flow](extension-to-runtime-capability-flow.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
