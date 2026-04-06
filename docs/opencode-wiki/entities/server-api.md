# Server API

## Overview

The server API exposes OpenCode as a Hono-based HTTP service with authentication, CORS, compression, OpenAPI documentation, global routes, workspace-aware routing, and websocket support. It is the service boundary that enables non-TUI clients.

The server is a first-class product surface that exposes the same runtime state machine behind the CLI.

In practice, this page should be read as a boundary explanation, not just a file list. Server API owns a specific slice of the OpenCode runtime, but it only makes sense in relation to neighboring systems such as [Control Plane and Workspaces](control-plane-and-workspaces.md), [MCP and ACP Integration](mcp-and-acp-integration.md), [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `packages/opencode/src/server/server.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `packages/opencode/src/server/router.ts`, `packages/opencode/src/server/routes/`, `packages/opencode/src/server/middleware.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Main Hono server construction and OpenAPI generation; Workspace-aware local/remote routing; Route handlers for sessions, files, projects, providers, pty, mcp, and more; Error and request middleware |
| Extension or policy points | The main extension or gating surfaces live around `packages/opencode/src/server/router.ts`, `packages/opencode/src/server/routes/`. |
| Persistent effects | Main Hono server construction and OpenAPI generation; Workspace-aware local/remote routing; Route handlers for sessions, files, projects, providers, pty, mcp, and more |

## Architecture

Server API is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `packages/opencode/src/server/server.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Server API does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `server/server.ts` becomes relevant when main Hono server construction and OpenAPI generation.
2. `server/router.ts` becomes relevant when workspace-aware local/remote routing.
3. `routes/` becomes relevant when route handlers for sessions, files, projects, providers, pty, mcp, and more.
4. `server/middleware.ts` becomes relevant when error and request middleware.
5. `server/projectors.ts` becomes relevant when server-side projector initialization.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `packages/opencode/src/server/router.ts`, `packages/opencode/src/server/routes/`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `packages/opencode/src/server/server.ts` | Main Hono server construction and OpenAPI generation |
| `packages/opencode/src/server/router.ts` | Workspace-aware local/remote routing |
| `packages/opencode/src/server/routes/` | Route handlers for sessions, files, projects, providers, pty, mcp, and more |
| `packages/opencode/src/server/middleware.ts` | Error and request middleware |
| `packages/opencode/src/server/projectors.ts` | Server-side projector initialization |

## See Also

- [Control Plane and Workspaces](control-plane-and-workspaces.md)
- [MCP and ACP Integration](mcp-and-acp-integration.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Request to Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
