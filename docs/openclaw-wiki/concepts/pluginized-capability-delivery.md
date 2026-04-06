# Pluginized Capability Delivery

## Overview

OpenClaw delivers a large share of its functionality through plugins and extensions: model providers, channels, memory behaviors, skills-adjacent integrations, browser/search/media capabilities, and various protocol or runtime add-ons.

This concept is a platform invariant. It explains why the gateway, routing, plugins, channels, and clients can evolve independently without collapsing into contradictory behavior. Pluginized Capability Delivery is therefore best read as a mechanism with operational consequences, not merely a label for related features.

## Why It Exists

This concept exists because the OpenClaw codebase repeatedly needs a stable way to coordinate behavior across multiple entities without turning those entities into one monolith.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

## Mechanism

A useful way to read this mechanism is as an ordered path through the participating subsystems:
1. Discover plugin roots and manifests.
2. Validate manifests and activation policy.
3. Build runtime APIs and registry state.
4. Activate plugin-contributed channels, providers, tools, hooks, or memory behavior.
5. Expose those capabilities through the gateway, CLI, and agent runtime.

The steps above are the operational skeleton. The exact file names vary by subsystem, but the concept remains stable because each participating entity contributes one predictable part of the chain. That is why the same concept can show up in SDK code, CLI wiring, plugin activation, channel routing, or persistence without becoming ambiguous.

## Invariants and Operational Implications

The most important invariant is that the linked entities are allowed to change implementation detail without changing the high-level guarantee described here. When a change breaks that guarantee, the failure usually appears at subsystem boundaries first: a summary no longer compacts correctly, a session route stops being stable, a skill path is not loaded consistently, or a permission rule is evaluated in the wrong layer.

Operationally, this means debugging should follow the mechanism rather than a UI symptom. Start where the concept is introduced, then inspect the next boundary where data, policy, or control is handed off. The source evidence table below is organized to support exactly that style of investigation.

## Involved Entities

| Entity | Role In This Concept |
| --- | --- |
| [Gateway as Control Plane](gateway-as-control-plane.md) | Why the gateway owns sessions, channels, tools, config, and UI surfaces |
| [Device Augmented Agent Architecture](device-augmented-agent-architecture.md) | Nodes, canvas, apps, and device execution as agent extensions |
| [Plugin Platform](../entities/plugin-platform.md) | Loader, manifests, install/uninstall, registry state, and runtime activation |
| [Extension to Runtime Capability Flow](../syntheses/extension-to-runtime-capability-flow.md) | How extensions become active providers, channels, or tools in the runtime |
| [Gateway As Control Plane](../concepts/gateway-as-control-plane.md) | Related page in this wiki. |
| [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md) | Related page in this wiki. |

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

- [Gateway as Control Plane](gateway-as-control-plane.md)
- [Device Augmented Agent Architecture](device-augmented-agent-architecture.md)
- [Plugin Platform](../entities/plugin-platform.md)
- [Extension to Runtime Capability Flow](../syntheses/extension-to-runtime-capability-flow.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
