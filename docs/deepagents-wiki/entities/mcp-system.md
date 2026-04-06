# MCP System

## Overview

The MCP system lets the CLI discover and connect Model Context Protocol servers, turning external tool providers into agent-usable tools. It supports user-level and project-level configuration files, plus multiple transport types.

MCP integration extends the default graph with externally hosted tools while preserving local trust decisions and CLI visibility.

In practice, this page should be read as a boundary explanation, not just a file list. MCP System owns a specific slice of the Deep Agents runtime, but it only makes sense in relation to neighboring systems such as [CLI Runtime](cli-runtime.md), [ACP Server](acp-server.md), [Local vs Remote Execution](../concepts/local-vs-remote-execution.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `libs/cli/deepagents_cli/mcp_tools.py` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `libs/cli/deepagents_cli/server_graph.py`, `libs/cli/deepagents_cli/mcp_trust.py` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | MCP config discovery, validation, and tool loading; Server-mode MCP preload and integration; Trust-policy helpers for project MCP |
| Extension or policy points | The main extension or gating surfaces live around `libs/cli/deepagents_cli/mcp_tools.py`, `libs/cli/deepagents_cli/mcp_trust.py`. |
| Persistent effects | Server-mode MCP preload and integration |

## Architecture

MCP System is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `libs/cli/deepagents_cli/mcp_tools.py`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what MCP System does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `deepagents_cli/mcp_tools.py` becomes relevant when mCP config discovery, validation, and tool loading.
2. `deepagents_cli/server_graph.py` becomes relevant when server-mode MCP preload and integration.
3. `deepagents_cli/mcp_trust.py` becomes relevant when trust-policy helpers for project MCP.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `libs/cli/deepagents_cli/mcp_tools.py`, `libs/cli/deepagents_cli/mcp_trust.py`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `libs/cli/deepagents_cli/mcp_tools.py` | MCP config discovery, validation, and tool loading |
| `libs/cli/deepagents_cli/server_graph.py` | Server-mode MCP preload and integration |
| `libs/cli/deepagents_cli/mcp_trust.py` | Trust-policy helpers for project MCP |

## See Also

- [CLI Runtime](cli-runtime.md)
- [ACP Server](acp-server.md)
- [Local vs Remote Execution](../concepts/local-vs-remote-execution.md)
- [SDK to CLI Composition](../syntheses/sdk-to-cli-composition.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
