# Channel System

## Overview

The channel system defines the common abstraction for all chat and messaging surfaces that OpenClaw supports. It normalizes channel IDs and metadata, carries session envelope information, applies mention/command gating, tracks typing and status reactions, and exposes helper APIs that the rest of the runtime can use without pulling in every concrete integration implementation.

Channel abstractions are what make many chat networks look like one assistant runtime without erasing network-specific policy.

In practice, this page should be read as a boundary explanation, not just a file list. Channel System owns a specific slice of the OpenClaw runtime, but it only makes sense in relation to neighboring systems such as [Channel Plugin Adapters](channel-plugin-adapters.md), [Routing System](routing-system.md), [Multi Channel Session Routing](../concepts/multi-channel-session-routing.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `src/channels/registry.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `src/channels/channel-config.ts`, `src/channels/mention-gating.ts`, `src/channels/command-gating.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Lightweight normalized channel lookup against active plugins; Shared channel configuration model; Mention-based activation control; Channel-specific command invocation policy |
| Extension or policy points | The main extension or gating surfaces live around `src/channels/registry.ts`, `src/channels/channel-config.ts`, `src/channels/command-gating.ts`. |
| Persistent effects | Channel/session association helpers |

## Architecture

Channel System is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `src/channels/registry.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Channel System does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `channels/registry.ts` becomes relevant when lightweight normalized channel lookup against active plugins.
2. `channels/channel-config.ts` becomes relevant when shared channel configuration model.
3. `channels/mention-gating.ts` becomes relevant when mention-based activation control.
4. `channels/command-gating.ts` becomes relevant when channel-specific command invocation policy.
5. `channels/session.ts` becomes relevant when channel/session association helpers.
6. `channels/typing.ts` becomes relevant when typing and typing-lifecycle behavior.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `src/channels/registry.ts`, `src/channels/channel-config.ts`, `src/channels/command-gating.ts`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `src/channels/registry.ts` | Lightweight normalized channel lookup against active plugins |
| `src/channels/channel-config.ts` | Shared channel configuration model |
| `src/channels/mention-gating.ts` | Mention-based activation control |
| `src/channels/command-gating.ts` | Channel-specific command invocation policy |
| `src/channels/session.ts` | Channel/session association helpers |
| `src/channels/typing.ts` | Typing and typing-lifecycle behavior |

## See Also

- [Channel Plugin Adapters](channel-plugin-adapters.md)
- [Routing System](routing-system.md)
- [Multi Channel Session Routing](../concepts/multi-channel-session-routing.md)
- [Inbound Message to Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
