# Control Plane and Workspaces

## Overview

The control plane and workspace subsystem lets OpenCode route requests either to local project instances or to remote workspaces behind adaptors. This is a major architectural differentiator relative to simpler single-process coding agents.

Workspaces let OpenCode talk about local and remote projects with one routing vocabulary.

In practice, this page should be read as a boundary explanation, not just a file list. Control Plane and Workspaces owns a specific slice of the OpenCode runtime, but it only makes sense in relation to neighboring systems such as [Server API](server-api.md), [Project and Instance System](project-and-instance-system.md), [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `packages/opencode/src/control-plane/workspace.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `packages/opencode/src/control-plane/schema.ts`, `packages/opencode/src/control-plane/adaptors/`, `packages/opencode/src/server/router.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Workspace records, adaptors, and sync loop; Workspace identifiers and schemas; Workspace adaptor implementations; Request routing between local and remote workspaces |
| Extension or policy points | The main extension or gating surfaces live around `packages/opencode/src/server/router.ts`. |
| Persistent effects | Workspace records, adaptors, and sync loop; Request routing between local and remote workspaces |

## Architecture

Control Plane and Workspaces is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `packages/opencode/src/control-plane/workspace.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Control Plane and Workspaces does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `control-plane/workspace.ts` becomes relevant when workspace records, adaptors, and sync loop.
2. `control-plane/schema.ts` becomes relevant when workspace identifiers and schemas.
3. `adaptors/` becomes relevant when workspace adaptor implementations.
4. `server/router.ts` becomes relevant when request routing between local and remote workspaces.
5. `control-plane/sse.ts` becomes relevant when sSE parsing and workspace event flow.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `packages/opencode/src/server/router.ts`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `packages/opencode/src/control-plane/workspace.ts` | Workspace records, adaptors, and sync loop |
| `packages/opencode/src/control-plane/schema.ts` | Workspace identifiers and schemas |
| `packages/opencode/src/control-plane/adaptors/` | Workspace adaptor implementations |
| `packages/opencode/src/server/router.ts` | Request routing between local and remote workspaces |
| `packages/opencode/src/control-plane/sse.ts` | SSE parsing and workspace event flow |

## See Also

- [Server API](server-api.md)
- [Project and Instance System](project-and-instance-system.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Workspace Routing and Remote Control](../syntheses/workspace-routing-and-remote-control.md)
- [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
