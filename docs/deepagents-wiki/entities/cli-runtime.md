# CLI Runtime

## Overview

The CLI runtime turns the SDK into a ready-to-run coding agent. It adds argument parsing, interactive and non-interactive execution modes, local context detection, session management, MCP loading, web search helpers, sandbox wiring, and CLI-specific approval logic.

The CLI is where the batteries-included SDK becomes a day-to-day product: config, approvals, sessions, and remote tooling all converge here.

In practice, this page should be read as a boundary explanation, not just a file list. CLI Runtime owns a specific slice of the Deep Agents runtime, but it only makes sense in relation to neighboring systems such as [Graph Factory](graph-factory.md), [CLI Textual UI](cli-textual-ui.md), [MCP System](mcp-system.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `libs/cli/deepagents_cli/main.py` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `libs/cli/deepagents_cli/agent.py`, `libs/cli/deepagents_cli/server_graph.py`, `libs/cli/deepagents_cli/tools.py` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Main CLI entry point and startup logic; CLI agent assembly and shell approval handling; Graph bootstrap for server mode; Web search and URL fetch helpers |
| Extension or policy points | The main extension or gating surfaces live around `libs/cli/deepagents_cli/agent.py`. |
| Persistent effects | CLI agent assembly and shell approval handling; Graph bootstrap for server mode |

## Architecture

CLI Runtime is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `libs/cli/deepagents_cli/main.py`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what CLI Runtime does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `deepagents_cli/main.py` becomes relevant when main CLI entry point and startup logic.
2. `deepagents_cli/agent.py` becomes relevant when cLI agent assembly and shell approval handling.
3. `deepagents_cli/server_graph.py` becomes relevant when graph bootstrap for server mode.
4. `deepagents_cli/tools.py` becomes relevant when web search and URL fetch helpers.
5. `deepagents_cli/local_context.py` becomes relevant when prompt-time local environment detection.
6. `cli/README.md` becomes relevant when product-level CLI description and feature summary.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `libs/cli/deepagents_cli/agent.py`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `libs/cli/deepagents_cli/main.py` | Main CLI entry point and startup logic |
| `libs/cli/deepagents_cli/agent.py` | CLI agent assembly and shell approval handling |
| `libs/cli/deepagents_cli/server_graph.py` | Graph bootstrap for server mode |
| `libs/cli/deepagents_cli/tools.py` | Web search and URL fetch helpers |
| `libs/cli/deepagents_cli/local_context.py` | Prompt-time local environment detection |
| `libs/cli/README.md` | Product-level CLI description and feature summary |

## See Also

- [Graph Factory](graph-factory.md)
- [CLI Textual UI](cli-textual-ui.md)
- [MCP System](mcp-system.md)
- [SDK to CLI Composition](../syntheses/sdk-to-cli-composition.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
