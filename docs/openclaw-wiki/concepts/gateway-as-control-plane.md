# Gateway as Control Plane

## Overview

The gateway is not just a transport server. It is the local control plane that coordinates sessions, channels, plugins, auth, cron, web surfaces, and external bridges.

This concept is a platform invariant. It explains why the gateway, routing, plugins, channels, and clients can evolve independently without collapsing into contradictory behavior. Gateway as Control Plane is therefore best read as a mechanism with operational consequences, not merely a label for related features.

## Why It Exists

This concept exists because the OpenClaw codebase repeatedly needs a stable way to coordinate behavior across multiple entities without turning those entities into one monolith.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

## Mechanism

A useful way to read this mechanism is as an ordered path through the participating subsystems:
1. Gateway startup loads config, secrets, plugins, and runtime state.
2. Channel managers, cron, node subscriptions, and websocket services are attached.
3. Session and transcript events are observed centrally.
4. External bridges like MCP and ACP connect through the gateway rather than bypassing it.
5. UI and management surfaces are served from the same control-plane runtime.

The steps above are the operational skeleton. The exact file names vary by subsystem, but the concept remains stable because each participating entity contributes one predictable part of the chain. That is why the same concept can show up in SDK code, CLI wiring, plugin activation, channel routing, or persistence without becoming ambiguous.

## Invariants and Operational Implications

The most important invariant is that the linked entities are allowed to change implementation detail without changing the high-level guarantee described here. When a change breaks that guarantee, the failure usually appears at subsystem boundaries first: a summary no longer compacts correctly, a session route stops being stable, a skill path is not loaded consistently, or a permission rule is evaluated in the wrong layer.

Operationally, this means debugging should follow the mechanism rather than a UI symptom. Start where the concept is introduced, then inspect the next boundary where data, policy, or control is handed off. The source evidence table below is organized to support exactly that style of investigation.

## Involved Entities

| Entity | Role In This Concept |
| --- | --- |
| [Local First Personal Assistant Architecture](local-first-personal-assistant-architecture.md) | OpenClaw as a user-owned assistant centered on a local gateway |
| [Pluginized Capability Delivery](pluginized-capability-delivery.md) | Providers, channels, and tools delivered through plugin activation |
| [Gateway Control Plane](../entities/gateway-control-plane.md) | Gateway server startup, auth, sessions, plugin bootstrap, and control-plane duties |
| [Inbound Message to Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md) | End-to-end flow from channel ingress through routing, session, agent, and reply dispatch |
| [Architecture Overview](../summaries/architecture-overview.md) | High-level orientation to OpenClaw as a local-first personal assistant platform |
| [Codebase Map](../summaries/codebase-map.md) | Maps `src/`, `extensions/`, `apps/`, `skills/`, and `docs/` to the corresponding wiki pages |

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `src/gateway/server.impl.ts` | Main composition root for gateway startup and subsystem wiring |
| `src/gateway/server-methods.ts` | Core control-plane method handlers |
| `src/plugins/loader.ts` | Central discovery, load, validation, and activation path |
| `src/plugins/runtime.ts` | Active registry lifecycle and runtime state |
| `src/sessions/session-id.ts` | Session ID validation helper |
| `src/sessions/session-lifecycle-events.ts` | Lifecycle event contracts for session creation and teardown |

## See Also

- [Local First Personal Assistant Architecture](local-first-personal-assistant-architecture.md)
- [Pluginized Capability Delivery](pluginized-capability-delivery.md)
- [Gateway Control Plane](../entities/gateway-control-plane.md)
- [Inbound Message to Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
