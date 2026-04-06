# Project and Instance System

## Overview

The project and instance system is the runtime scoping layer. It determines which directory and worktree an operation belongs to, initializes project-local services, and ensures cleanup happens when the instance is disposed or reloaded.

Project scoping in OpenCode creates a durable runtime boundary that client surfaces can share.

In practice, this page should be read as a boundary explanation, not just a file list. Project and Instance System owns a specific slice of the OpenCode runtime, but it only makes sense in relation to neighboring systems such as [Control Plane and Workspaces](control-plane-and-workspaces.md), [Plugin System](plugin-system.md), [Project Scoped Instance Lifecycle](../concepts/project-scoped-instance-lifecycle.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `packages/opencode/src/project/instance.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `packages/opencode/src/project/bootstrap.ts`, `packages/opencode/src/project/project.ts`, `packages/opencode/src/project/vcs.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Project-scoped instance context, caching, reload, and disposal; Bootstrap sequence for plugins, file watchers, VCS, and snapshots; Project discovery and metadata; VCS integration |
| Extension or policy points | The main extension or gating surfaces live around `packages/opencode/src/project/bootstrap.ts`. |
| Persistent effects | state changes and runtime handoffs described in the cited files |

## Architecture

Project and Instance System is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `packages/opencode/src/project/instance.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Project and Instance System does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `project/instance.ts` becomes relevant when project-scoped instance context, caching, reload, and disposal.
2. `project/bootstrap.ts` becomes relevant when bootstrap sequence for plugins, file watchers, VCS, and snapshots.
3. `project/project.ts` becomes relevant when project discovery and metadata.
4. `project/vcs.ts` becomes relevant when vCS integration.
5. `project/state.ts` becomes relevant when project-scoped state management.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `packages/opencode/src/project/bootstrap.ts`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `packages/opencode/src/project/instance.ts` | Project-scoped instance context, caching, reload, and disposal |
| `packages/opencode/src/project/bootstrap.ts` | Bootstrap sequence for plugins, file watchers, VCS, and snapshots |
| `packages/opencode/src/project/project.ts` | Project discovery and metadata |
| `packages/opencode/src/project/vcs.ts` | VCS integration |
| `packages/opencode/src/project/state.ts` | Project-scoped state management |

## See Also

- [Control Plane and Workspaces](control-plane-and-workspaces.md)
- [Plugin System](plugin-system.md)
- [Project Scoped Instance Lifecycle](../concepts/project-scoped-instance-lifecycle.md)
- [Workspace Routing and Remote Control](../syntheses/workspace-routing-and-remote-control.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
