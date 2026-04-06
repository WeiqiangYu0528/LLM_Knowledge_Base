# Canvas and Control UI

## Overview

Canvas and the control UI are OpenClaw's visual surfaces. They make the assistant more than a chat endpoint by hosting an interactive workspace, serving live UI assets, and bridging user actions back into the runtime. The gateway serves these surfaces as part of the control plane, while the canvas host handles file serving, websocket live reload, and A2UI-specific request handling.

Visual surfaces are part of the control plane, not a disconnected frontend.

In practice, this page should be read as a boundary explanation, not just a file list. Canvas and Control UI owns a specific slice of the OpenClaw runtime, but it only makes sense in relation to neighboring systems such as [Node Host and Device Pairing](node-host-and-device-pairing.md), [Native Apps and Platform Clients](native-apps-and-platform-clients.md), [Device Augmented Agent Architecture](../concepts/device-augmented-agent-architecture.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `src/canvas-host/server.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `src/canvas-host/a2ui.ts`, `src/canvas-host/file-resolver.ts`, `src/gateway/control-ui-http-utils.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Canvas HTTP/WebSocket host implementation; A2UI request handling and live-reload injection; Safe path resolution inside canvas roots; Control UI HTTP helper functions |
| Extension or policy points | The main extension or gating surfaces live around the neighboring modules listed in the source map. |
| Persistent effects | Canvas HTTP/WebSocket host implementation |

## Architecture

Canvas and Control UI is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `src/canvas-host/server.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Canvas and Control UI does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `canvas-host/server.ts` becomes relevant when canvas HTTP/WebSocket host implementation.
2. `canvas-host/a2ui.ts` becomes relevant when a2UI request handling and live-reload injection.
3. `canvas-host/file-resolver.ts` becomes relevant when safe path resolution inside canvas roots.
4. `gateway/control-ui-http-utils.ts` becomes relevant when control UI HTTP helper functions.
5. `gateway/control-ui-shared.ts` becomes relevant when shared control UI behavior and state helpers.
6. `ui/` becomes relevant when frontend/control UI codebase.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around the neighboring modules listed in the source map. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `src/canvas-host/server.ts` | Canvas HTTP/WebSocket host implementation |
| `src/canvas-host/a2ui.ts` | A2UI request handling and live-reload injection |
| `src/canvas-host/file-resolver.ts` | Safe path resolution inside canvas roots |
| `src/gateway/control-ui-http-utils.ts` | Control UI HTTP helper functions |
| `src/gateway/control-ui-shared.ts` | Shared control UI behavior and state helpers |
| `ui/` | Frontend/control UI codebase |

## See Also

- [Node Host and Device Pairing](node-host-and-device-pairing.md)
- [Native Apps and Platform Clients](native-apps-and-platform-clients.md)
- [Device Augmented Agent Architecture](../concepts/device-augmented-agent-architecture.md)
- [Canvas Voice and Device Control Loop](../syntheses/canvas-voice-and-device-control-loop.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
