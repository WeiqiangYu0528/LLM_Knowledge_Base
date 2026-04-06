# Channel Plugin Adapters

## Overview

Channel plugin adapters are the concrete integration layer that connects OpenClaw to messaging networks and channel-like surfaces. They handle configured bindings, pairing/setup flows, account state, target resolution, approvals, and runtime registration for integrations such as Slack, Telegram, Discord, Signal, WhatsApp, and many others.

Adapters are where plugin-provided capability becomes concrete ingress and egress behavior.

In practice, this page should be read as a boundary explanation, not just a file list. Channel Plugin Adapters owns a specific slice of the OpenClaw runtime, but it only makes sense in relation to neighboring systems such as [Channel System](channel-system.md), [Plugin Platform](plugin-platform.md), [Pluginized Capability Delivery](../concepts/pluginized-capability-delivery.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `src/channels/plugins/index.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `src/channels/plugins/load.ts`, `src/channels/plugins/bootstrap-registry.ts`, `src/channels/plugins/setup-registry.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Main export surface for channel plugin integration; Channel plugin load path; Bootstrap-time registration; Setup-time registry management |
| Extension or policy points | The main extension or gating surfaces live around `src/channels/plugins/index.ts`, `src/channels/plugins/load.ts`, `src/channels/plugins/bootstrap-registry.ts`, `src/channels/plugins/setup-registry.ts`. |
| Persistent effects | state changes and runtime handoffs described in the cited files |

## Architecture

Channel Plugin Adapters is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `src/channels/plugins/index.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Channel Plugin Adapters does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `plugins/index.ts` becomes relevant when main export surface for channel plugin integration.
2. `plugins/load.ts` becomes relevant when channel plugin load path.
3. `plugins/bootstrap-registry.ts` becomes relevant when bootstrap-time registration.
4. `plugins/setup-registry.ts` becomes relevant when setup-time registry management.
5. `plugins/target-resolvers.ts` becomes relevant when resolution of outbound/inbound channel targets.
6. `plugins/pairing.ts` becomes relevant when pairing workflows for channel/device-like integrations.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `src/channels/plugins/index.ts`, `src/channels/plugins/load.ts`, `src/channels/plugins/bootstrap-registry.ts`, `src/channels/plugins/setup-registry.ts`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `src/channels/plugins/index.ts` | Main export surface for channel plugin integration |
| `src/channels/plugins/load.ts` | Channel plugin load path |
| `src/channels/plugins/bootstrap-registry.ts` | Bootstrap-time registration |
| `src/channels/plugins/setup-registry.ts` | Setup-time registry management |
| `src/channels/plugins/target-resolvers.ts` | Resolution of outbound/inbound channel targets |
| `src/channels/plugins/pairing.ts` | Pairing workflows for channel/device-like integrations |
| `src/channels/plugins/approvals.ts` | Approval surfaces and enforcement hooks |

## See Also

- [Channel System](channel-system.md)
- [Plugin Platform](plugin-platform.md)
- [Pluginized Capability Delivery](../concepts/pluginized-capability-delivery.md)
- [Channel Binding and Session Identity Flow](../syntheses/channel-binding-and-session-identity-flow.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
