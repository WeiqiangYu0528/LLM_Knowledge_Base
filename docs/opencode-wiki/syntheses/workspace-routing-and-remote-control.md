# Workspace Routing and Remote Control

## Overview

This synthesis explains how OpenCode routes requests between local project instances and remote workspaces, making remote control and distributed execution part of the product architecture.

This synthesis matters because OpenCode is multi-surface; the runtime only makes sense when the handoff between client, session, provider, tool, and persistence layers is explicit. The point of this page is to make the handoff between systems explicit so a reader can reason about failures, extension points, and policy boundaries without re-deriving the whole path from raw source.

## Systems Involved

| System | Contribution |
| --- | --- |
| [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md) | OpenCode core service with multiple clients layered on top |
| [Control Plane and Workspaces](../entities/control-plane-and-workspaces.md) | Workspace model, adaptors, remote routing, and SSE syncing |
| [Server API](../entities/server-api.md) | Hono server, middleware, OpenAPI generation, and routed APIs |
| [Multi Client Product Architecture](multi-client-product-architecture.md) | TUI, app, web, desktop, Slack, and SDK surfaces around one core |
| [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md) | Related page in this wiki. |
| [Architecture Overview](../summaries/architecture-overview.md) | High-level orientation to the OpenCode monorepo, package boundaries, and runtime model |

## Interaction Model

Read this interaction model as a boundary-by-boundary path rather than a single function call:
1. [Control Plane and Workspaces](../entities/control-plane-and-workspaces.md) contributes one stage of the cross-system path described by this synthesis.
2. [Server API](../entities/server-api.md) contributes one stage of the cross-system path described by this synthesis.
3. [Project and Instance System](../entities/project-and-instance-system.md) contributes one stage of the cross-system path described by this synthesis.
4. [MCP and ACP Integration](../entities/mcp-and-acp-integration.md) contributes one stage of the cross-system path described by this synthesis.

What matters here is where ownership changes hands. One subsystem usually receives input, another owns durable state, another chooses policy or execution strategy, and another surfaces the result back to a user or peer system. If those boundaries are not visible, the runtime looks simpler than it actually is and changes become riskier.

## Key Interfaces

| Interface Or Contract | Why It Matters |
| --- | --- |
| `Workspace records, adaptors, and sync loop` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Workspace identifiers and schemas` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Main Hono server construction and OpenAPI generation` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Workspace-aware local/remote routing` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Project-scoped instance context, caching, reload, and disposal` | Boundary surface called out by the current page or inferred from the cited sources. |

## Operational Consequences

The synthesis view is usually where debugging stops being guesswork. A symptom that appears in the UI or CLI often originates several layers deeper, especially in repos that separate session state, routing, protocol bridging, plugin activation, and persistence. By following the sequence above, an implementer can decide whether a bug belongs in the initiating surface, the policy layer, the execution layer, or the state-reconciliation layer.

This also explains why change review should span more than one page. A modification that looks local inside an entity page may still alter a synthesis if it changes when state is persisted, which system chooses the active model, how approvals are routed, or where an outbound reply is emitted.

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `packages/opencode/src/control-plane/workspace.ts` | Workspace records, adaptors, and sync loop |
| `packages/opencode/src/control-plane/schema.ts` | Workspace identifiers and schemas |
| `packages/opencode/src/server/server.ts` | Main Hono server construction and OpenAPI generation |
| `packages/opencode/src/server/router.ts` | Workspace-aware local/remote routing |
| `packages/opencode/src/project/instance.ts` | Project-scoped instance context, caching, reload, and disposal |
| `packages/opencode/src/project/bootstrap.ts` | Bootstrap sequence for plugins, file watchers, VCS, and snapshots |

## See Also

- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Control Plane and Workspaces](../entities/control-plane-and-workspaces.md)
- [Server API](../entities/server-api.md)
- [Multi Client Product Architecture](multi-client-product-architecture.md)
- [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
