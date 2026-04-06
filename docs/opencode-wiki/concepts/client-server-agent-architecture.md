# Client Server Agent Architecture

## Overview

OpenCode is built around the idea that the coding agent runtime should be reachable from multiple clients. The README makes this explicit: the TUI is only one possible client, and the same engine can be driven remotely.

This concept exists because OpenCode reuses one runtime across multiple entry points. The concept page therefore needs to explain the shared mechanism that keeps those surfaces aligned. Client Server Agent Architecture is therefore best read as a mechanism with operational consequences, not merely a label for related features.

## Why It Exists

This concept exists because the OpenCode codebase repeatedly needs a stable way to coordinate behavior across multiple entities without turning those entities into one monolith.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

## Mechanism

A useful way to read this mechanism is as an ordered path through the participating subsystems:
1. a core runtime inside `packages/opencode/src`.
2. a server API built with Hono.
3. workspace-aware routing between local and remote instances.
4. multiple client packages consuming the same runtime indirectly.
5. The architecture emerges from several cooperating pieces: 1.
6. a core runtime inside `packages/opencode/src` 2.
7. a server API built with Hono 3.
8. workspace-aware routing between local and remote instances 4.

The steps above are the operational skeleton. The exact file names vary by subsystem, but the concept remains stable because each participating entity contributes one predictable part of the chain. That is why the same concept can show up in SDK code, CLI wiring, plugin activation, channel routing, or persistence without becoming ambiguous.

## Invariants and Operational Implications

The most important invariant is that the linked entities are allowed to change implementation detail without changing the high-level guarantee described here. When a change breaks that guarantee, the failure usually appears at subsystem boundaries first: a summary no longer compacts correctly, a session route stops being stable, a skill path is not loaded consistently, or a permission rule is evaluated in the wrong layer.

Operationally, this means debugging should follow the mechanism rather than a UI symptom. Start where the concept is introduced, then inspect the next boundary where data, policy, or control is handed off. The source evidence table below is organized to support exactly that style of investigation.

## Involved Entities

| Entity | Role In This Concept |
| --- | --- |
| [Workspace Routing and Remote Control](../syntheses/workspace-routing-and-remote-control.md) | Local vs remote workspaces, adaptors, and forwarded requests |
| [Multi Client Product Architecture](../syntheses/multi-client-product-architecture.md) | TUI, app, web, desktop, Slack, and SDK surfaces around one core |
| [Server API](../entities/server-api.md) | Hono server, middleware, OpenAPI generation, and routed APIs |
| [Control Plane and Workspaces](../entities/control-plane-and-workspaces.md) | Workspace model, adaptors, remote routing, and SSE syncing |
| [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md) | Related page in this wiki. |
| [Architecture Overview](../summaries/architecture-overview.md) | High-level orientation to the OpenCode monorepo, package boundaries, and runtime model |

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `packages/opencode/src/index.ts` | Root command entry point and startup/migration flow |
| `packages/opencode/src/cli/bootstrap.ts` | Project-scoped bootstrap helper |
| `packages/opencode/src/server/server.ts` | Main Hono server construction and OpenAPI generation |
| `packages/opencode/src/server/router.ts` | Workspace-aware local/remote routing |
| `packages/opencode/src/control-plane/workspace.ts` | Workspace records, adaptors, and sync loop |
| `packages/opencode/src/control-plane/schema.ts` | Workspace identifiers and schemas |

## See Also

- [Workspace Routing and Remote Control](../syntheses/workspace-routing-and-remote-control.md)
- [Multi Client Product Architecture](../syntheses/multi-client-product-architecture.md)
- [Server API](../entities/server-api.md)
- [Control Plane and Workspaces](../entities/control-plane-and-workspaces.md)
- [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
