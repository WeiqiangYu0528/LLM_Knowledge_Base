# UI Client Surfaces

## Overview

OpenCode's app, UI, and web packages represent the user-facing product surfaces built on top of the core runtime. The shared UI package is especially large, indicating a serious investment in front-end presentation and reusable components.

Client packages matter because they consume the core runtime as a platform, not because they own separate agent loops.

In practice, this page should be read as a boundary explanation, not just a file list. UI Client Surfaces owns a specific slice of the OpenCode runtime, but it only makes sense in relation to neighboring systems such as [Desktop and Remote Clients](desktop-and-remote-clients.md), [Server API](server-api.md), [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `packages/app/package.json` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `packages/app/README.md`, `packages/ui/package.json`, `packages/web/package.json` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Main app client package and scripts; App usage and e2e notes; Shared UI/component package; Web/docs package |
| Extension or policy points | The main extension or gating surfaces live around the neighboring modules listed in the source map. |
| Persistent effects | state changes and runtime handoffs described in the cited files |

## Architecture

UI Client Surfaces is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `packages/app/package.json`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what UI Client Surfaces does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `app/package.json` becomes relevant when main app client package and scripts.
2. `app/README.md` becomes relevant when app usage and e2e notes.
3. `ui/package.json` becomes relevant when shared UI/component package.
4. `web/package.json` becomes relevant when web/docs package.
5. `web/README.md` becomes relevant when web/starlight package notes.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around the neighboring modules listed in the source map. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `packages/app/package.json` | Main app client package and scripts |
| `packages/app/README.md` | App usage and e2e notes |
| `packages/ui/package.json` | Shared UI/component package |
| `packages/web/package.json` | Web/docs package |
| `packages/web/README.md` | Web/starlight package notes |

## See Also

- [Desktop and Remote Clients](desktop-and-remote-clients.md)
- [Server API](server-api.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Multi Client Product Architecture](../syntheses/multi-client-product-architecture.md)
- [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
