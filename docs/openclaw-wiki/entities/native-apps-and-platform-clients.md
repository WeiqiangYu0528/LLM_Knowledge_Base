# Native Apps and Platform Clients

## Overview

OpenClaw ships beyond the CLI. The repo includes macOS, iOS, Android, and shared app surfaces, plus web/control UI components and protocol clients. These are not optional demos; they are part of the product's device presence and help make OpenClaw feel like a persistent personal assistant rather than only a terminal tool.

Native apps are client surfaces over the gateway-centric runtime rather than isolated assistant implementations.

In practice, this page should be read as a boundary explanation, not just a file list. Native Apps and Platform Clients owns a specific slice of the OpenClaw runtime, but it only makes sense in relation to neighboring systems such as [Canvas and Control UI](canvas-and-control-ui.md), [Media and Voice Stack](media-and-voice-stack.md), [Device Augmented Agent Architecture](../concepts/device-augmented-agent-architecture.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `apps/macos/` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `apps/ios/`, `apps/android/`, `apps/shared/` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | macOS client and platform-specific integration; iOS client and node-like mobile features; Android client and device-command surface; Shared client logic/assets |
| Extension or policy points | The main extension or gating surfaces live around the neighboring modules listed in the source map. |
| Persistent effects | state changes and runtime handoffs described in the cited files |

## Architecture

Native Apps and Platform Clients is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `apps/macos/`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Native Apps and Platform Clients does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `macos/` becomes relevant when macOS client and platform-specific integration.
2. `ios/` becomes relevant when iOS client and node-like mobile features.
3. `android/` becomes relevant when android client and device-command surface.
4. `shared/` becomes relevant when shared client logic/assets.
5. `ui/` becomes relevant when web/control UI frontend surface.
6. `README.md` becomes relevant when high-level description of app and device capabilities.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around the neighboring modules listed in the source map. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `apps/macos/` | macOS client and platform-specific integration |
| `apps/ios/` | iOS client and node-like mobile features |
| `apps/android/` | Android client and device-command surface |
| `apps/shared/` | Shared client logic/assets |
| `ui/` | Web/control UI frontend surface |
| `README.md` | High-level description of app and device capabilities |

## See Also

- [Canvas and Control UI](canvas-and-control-ui.md)
- [Media and Voice Stack](media-and-voice-stack.md)
- [Device Augmented Agent Architecture](../concepts/device-augmented-agent-architecture.md)
- [Canvas Voice and Device Control Loop](../syntheses/canvas-voice-and-device-control-loop.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
