# MCP and ACP Integration

## Overview

OpenCode supports both Model Context Protocol and Agent Client Protocol. MCP expands the tool/resource surface by connecting to external MCP servers; ACP exposes OpenCode itself as an editor-hosted agent.

This subsystem matters because mcp client management and acp editor-facing agent bridge.

In practice, this page should be read as a boundary explanation, not just a file list. MCP and ACP Integration owns a specific slice of the OpenCode runtime, but it only makes sense in relation to neighboring systems such as [Server API](server-api.md), [Provider System](provider-system.md), [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `packages/opencode/src/mcp/index.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `packages/opencode/src/mcp/auth.ts`, `packages/opencode/src/mcp/oauth-provider.ts`, `packages/opencode/src/acp/agent.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | MCP client/service implementation, tool conversion, auth, and status tracking; MCP auth support; OAuth provider integration for MCP; ACP agent bridge and permission/update forwarding |
| Extension or policy points | The main extension or gating surfaces live around `packages/opencode/src/mcp/index.ts`, `packages/opencode/src/mcp/auth.ts`, `packages/opencode/src/mcp/oauth-provider.ts`. |
| Persistent effects | ACP session management |

## Architecture

MCP and ACP Integration is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `packages/opencode/src/mcp/index.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what MCP and ACP Integration does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `mcp/index.ts` becomes relevant when mCP client/service implementation, tool conversion, auth, and status tracking.
2. `mcp/auth.ts` becomes relevant when mCP auth support.
3. `mcp/oauth-provider.ts` becomes relevant when oAuth provider integration for MCP.
4. `acp/agent.ts` becomes relevant when aCP agent bridge and permission/update forwarding.
5. `acp/session.ts` becomes relevant when aCP session management.
6. `acp/README.md` becomes relevant when aCP integration notes.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `packages/opencode/src/mcp/index.ts`, `packages/opencode/src/mcp/auth.ts`, `packages/opencode/src/mcp/oauth-provider.ts`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `packages/opencode/src/mcp/index.ts` | MCP client/service implementation, tool conversion, auth, and status tracking |
| `packages/opencode/src/mcp/auth.ts` | MCP auth support |
| `packages/opencode/src/mcp/oauth-provider.ts` | OAuth provider integration for MCP |
| `packages/opencode/src/acp/agent.ts` | ACP agent bridge and permission/update forwarding |
| `packages/opencode/src/acp/session.ts` | ACP session management |
| `packages/opencode/src/acp/README.md` | ACP integration notes |

## See Also

- [Server API](server-api.md)
- [Provider System](provider-system.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Workspace Routing and Remote Control](../syntheses/workspace-routing-and-remote-control.md)
- [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
