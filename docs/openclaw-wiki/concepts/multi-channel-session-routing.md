# Multi Channel Session Routing

## Overview

OpenClaw's routing model exists because one assistant can appear across many channels, accounts, peers, and group contexts. The platform therefore needs a deterministic way to map inbound traffic into isolated agent/session lanes.

This concept is a platform invariant. It explains why the gateway, routing, plugins, channels, and clients can evolve independently without collapsing into contradictory behavior. Multi Channel Session Routing is therefore best read as a mechanism with operational consequences, not merely a label for related features.

## Why It Exists

This concept exists because the OpenClaw codebase repeatedly needs a stable way to coordinate behavior across multiple entities without turning those entities into one monolith.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

## Mechanism

A useful way to read this mechanism is as an ordered path through the participating subsystems:
1. Normalize channel and account information.
2. Evaluate bindings, peer wildcards, guild/team scope, and role-sensitive rules.
3. Resolve the effective `agentId` and `accountId`.
4. Construct `sessionKey` and `mainSessionKey`.
5. Hand the resolved route to session, transcript, and reply systems.

The steps above are the operational skeleton. The exact file names vary by subsystem, but the concept remains stable because each participating entity contributes one predictable part of the chain. That is why the same concept can show up in SDK code, CLI wiring, plugin activation, channel routing, or persistence without becoming ambiguous.

## Invariants and Operational Implications

The most important invariant is that the linked entities are allowed to change implementation detail without changing the high-level guarantee described here. When a change breaks that guarantee, the failure usually appears at subsystem boundaries first: a summary no longer compacts correctly, a session route stops being stable, a skill path is not loaded consistently, or a permission rule is evaluated in the wrong layer.

Operationally, this means debugging should follow the mechanism rather than a UI symptom. Start where the concept is introduced, then inspect the next boundary where data, policy, or control is handed off. The source evidence table below is organized to support exactly that style of investigation.

## Involved Entities

| Entity | Role In This Concept |
| --- | --- |
| [Auth and Approval Boundaries](auth-and-approval-boundaries.md) | Tokens, passwords, allowlists, approvals, and safety envelopes |
| [Routing System](../entities/routing-system.md) | Account, peer, and binding-based route resolution into agent/session targets |
| [Session System](../entities/session-system.md) | Session identity, lifecycle, provenance, send policy, and transcript events |
| [Channel Binding and Session Identity Flow](../syntheses/channel-binding-and-session-identity-flow.md) | How bindings, account IDs, routes, and session keys compose |
| [Gateway As Control Plane](../concepts/gateway-as-control-plane.md) | Related page in this wiki. |
| [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md) | Related page in this wiki. |

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `src/routing/resolve-route.ts` | Main route resolution logic |
| `src/routing/session-key.ts` | Session key construction and normalization helpers |
| `src/sessions/session-id.ts` | Session ID validation helper |
| `src/sessions/session-lifecycle-events.ts` | Lifecycle event contracts for session creation and teardown |
| `src/channels/registry.ts` | Lightweight normalized channel lookup against active plugins |
| `src/channels/channel-config.ts` | Shared channel configuration model |

## See Also

- [Auth and Approval Boundaries](auth-and-approval-boundaries.md)
- [Routing System](../entities/routing-system.md)
- [Session System](../entities/session-system.md)
- [Channel Binding and Session Identity Flow](../syntheses/channel-binding-and-session-identity-flow.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
