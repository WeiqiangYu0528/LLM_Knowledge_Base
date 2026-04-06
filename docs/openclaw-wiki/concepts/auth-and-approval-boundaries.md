# Auth and Approval Boundaries

## Overview

OpenClaw crosses many trust boundaries: gateway auth, remote exposure, channel credentials, provider credentials, node/device pairing, and command or execution approvals. The repo repeatedly separates identity, authorization, and approval policy instead of assuming one broad trust zone.

This concept is a platform invariant. It explains why the gateway, routing, plugins, channels, and clients can evolve independently without collapsing into contradictory behavior. Auth and Approval Boundaries is therefore best read as a mechanism with operational consequences, not merely a label for related features.

## Why It Exists

This concept exists because the OpenClaw codebase repeatedly needs a stable way to coordinate behavior across multiple entities without turning those entities into one monolith.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

## Mechanism

A useful way to read this mechanism is as an ordered path through the participating subsystems:
1. Resolve gateway connection credentials and auth mode.
2. Apply plugin/provider/channel-specific auth logic.
3. Pair devices or nodes with explicit identity and capability metadata.
4. Gate risky actions through approvals, allowlists, or command policies.
5. Keep session and route state distinct from raw credential material.

The steps above are the operational skeleton. The exact file names vary by subsystem, but the concept remains stable because each participating entity contributes one predictable part of the chain. That is why the same concept can show up in SDK code, CLI wiring, plugin activation, channel routing, or persistence without becoming ambiguous.

## Invariants and Operational Implications

The most important invariant is that the linked entities are allowed to change implementation detail without changing the high-level guarantee described here. When a change breaks that guarantee, the failure usually appears at subsystem boundaries first: a summary no longer compacts correctly, a session route stops being stable, a skill path is not loaded consistently, or a permission rule is evaluated in the wrong layer.

Operationally, this means debugging should follow the mechanism rather than a UI symptom. Start where the concept is introduced, then inspect the next boundary where data, policy, or control is handed off. The source evidence table below is organized to support exactly that style of investigation.

## Involved Entities

| Entity | Role In This Concept |
| --- | --- |
| [Multi Channel Session Routing](multi-channel-session-routing.md) | How inbound channels and accounts map into isolated sessions and agents |
| [Device Augmented Agent Architecture](device-augmented-agent-architecture.md) | Nodes, canvas, apps, and device execution as agent extensions |
| [Gateway Control Plane](../entities/gateway-control-plane.md) | Gateway server startup, auth, sessions, plugin bootstrap, and control-plane duties |
| [Scheduled Job Delivery Flow](../syntheses/scheduled-job-delivery-flow.md) | How cron normalization, isolated execution, delivery, and cleanup compose |
| [Gateway As Control Plane](../concepts/gateway-as-control-plane.md) | Related page in this wiki. |
| [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md) | Related page in this wiki. |

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `src/gateway/server.impl.ts` | Main composition root for gateway startup and subsystem wiring |
| `src/gateway/server-methods.ts` | Core control-plane method handlers |
| `src/plugins/provider-catalog.ts` | Provider inventory and catalog behavior |
| `src/plugins/providers.runtime.ts` | Runtime-facing provider assembly |
| `src/channels/plugins/index.ts` | Main export surface for channel plugin integration |
| `src/channels/plugins/load.ts` | Channel plugin load path |

## See Also

- [Multi Channel Session Routing](multi-channel-session-routing.md)
- [Device Augmented Agent Architecture](device-augmented-agent-architecture.md)
- [Gateway Control Plane](../entities/gateway-control-plane.md)
- [Scheduled Job Delivery Flow](../syntheses/scheduled-job-delivery-flow.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
