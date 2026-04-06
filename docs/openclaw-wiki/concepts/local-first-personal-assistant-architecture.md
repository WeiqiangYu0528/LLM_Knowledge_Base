# Local First Personal Assistant Architecture

## Overview

OpenClaw is organized around the idea that a personal assistant should run on the user's own devices and accounts rather than primarily inside a hosted SaaS shell. That principle shapes the gateway, node pairing, channel integrations, companion apps, and the filesystem-heavy setup model.

This concept is a platform invariant. It explains why the gateway, routing, plugins, channels, and clients can evolve independently without collapsing into contradictory behavior. Local First Personal Assistant Architecture is therefore best read as a mechanism with operational consequences, not merely a label for related features.

## Why It Exists

This concept exists because the OpenClaw codebase repeatedly needs a stable way to coordinate behavior across multiple entities without turning those entities into one monolith.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

## Mechanism

A useful way to read this mechanism is as an ordered path through the participating subsystems:
1. A local gateway owns configuration, sessions, tools, channels, and UI surfaces.
2. Channels and apps connect to that gateway rather than to an external monolithic service.
3. Device or node clients extend the assistant into phones, desktops, and local execution surfaces.
4. Plugins and skills let the assistant adapt to user-specific providers, channels, and workflows.
5. The pattern works like this: 1.

The steps above are the operational skeleton. The exact file names vary by subsystem, but the concept remains stable because each participating entity contributes one predictable part of the chain. That is why the same concept can show up in SDK code, CLI wiring, plugin activation, channel routing, or persistence without becoming ambiguous.

## Invariants and Operational Implications

The most important invariant is that the linked entities are allowed to change implementation detail without changing the high-level guarantee described here. When a change breaks that guarantee, the failure usually appears at subsystem boundaries first: a summary no longer compacts correctly, a session route stops being stable, a skill path is not loaded consistently, or a permission rule is evaluated in the wrong layer.

Operationally, this means debugging should follow the mechanism rather than a UI symptom. Start where the concept is introduced, then inspect the next boundary where data, policy, or control is handed off. The source evidence table below is organized to support exactly that style of investigation.

## Involved Entities

| Entity | Role In This Concept |
| --- | --- |
| [Gateway as Control Plane](gateway-as-control-plane.md) | Why the gateway owns sessions, channels, tools, config, and UI surfaces |
| [Device Augmented Agent Architecture](device-augmented-agent-architecture.md) | Nodes, canvas, apps, and device execution as agent extensions |
| [Gateway Control Plane](../entities/gateway-control-plane.md) | Gateway server startup, auth, sessions, plugin bootstrap, and control-plane duties |
| [Onboarding to Live Gateway Flow](../syntheses/onboarding-to-live-gateway-flow.md) | Install, configure, daemonize, and bring a live OpenClaw gateway online |
| [Gateway As Control Plane](../concepts/gateway-as-control-plane.md) | Related page in this wiki. |
| [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md) | Related page in this wiki. |

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `src/gateway/server.impl.ts` | Main composition root for gateway startup and subsystem wiring |
| `src/gateway/server-methods.ts` | Core control-plane method handlers |
| `src/cli/run-main.ts` | Top-level CLI bootstrap, env normalization, and command routing |
| `src/cli/program.ts` | Commander program construction |
| `src/node-host/runner.ts` | Main node-host process and gateway event loop |
| `src/node-host/invoke.ts` | Invocation handling |

## See Also

- [Gateway as Control Plane](gateway-as-control-plane.md)
- [Device Augmented Agent Architecture](device-augmented-agent-architecture.md)
- [Gateway Control Plane](../entities/gateway-control-plane.md)
- [Onboarding to Live Gateway Flow](../syntheses/onboarding-to-live-gateway-flow.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
