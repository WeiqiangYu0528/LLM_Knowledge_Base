# Subagent System

## Overview

The subagent system gives Deep Agents an explicit delegation mechanism instead of expecting a single context window to do everything. It exposes a `task` tool backed by subagent specifications, compiled runnables, or remote async agents.

Delegation in Deep Agents is a first-class part of the graph rather than an afterthought added only in the CLI.

In practice, this page should be read as a boundary explanation, not just a file list. Subagent System owns a specific slice of the Deep Agents runtime, but it only makes sense in relation to neighboring systems such as [Graph Factory](graph-factory.md), [CLI Runtime](cli-runtime.md), [Filesystem First Agent Configuration](../concepts/filesystem-first-agent-configuration.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `libs/deepagents/deepagents/middleware/subagents.py` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `libs/deepagents/deepagents/middleware/async_subagents.py`, `libs/cli/deepagents_cli/subagents.py`, `libs/cli/deepagents_cli/agent.py` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Core subagent specs, tool description, and middleware integration; Remote/background subagent handling; CLI loader for filesystem-defined subagents; Loads async subagent definitions from config and passes them into CLI agent creation |
| Extension or policy points | The main extension or gating surfaces live around `libs/deepagents/deepagents/middleware/async_subagents.py`, `libs/cli/deepagents_cli/agent.py`. |
| Persistent effects | Remote/background subagent handling; Loads async subagent definitions from config and passes them into CLI agent creation |

## Architecture

Subagent System is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `libs/deepagents/deepagents/middleware/subagents.py`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Subagent System does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `middleware/subagents.py` becomes relevant when core subagent specs, tool description, and middleware integration.
2. `middleware/async_subagents.py` becomes relevant when remote/background subagent handling.
3. `deepagents_cli/subagents.py` becomes relevant when cLI loader for filesystem-defined subagents.
4. `deepagents_cli/agent.py` becomes relevant when loads async subagent definitions from config and passes them into CLI agent creation.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `libs/deepagents/deepagents/middleware/async_subagents.py`, `libs/cli/deepagents_cli/agent.py`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `libs/deepagents/deepagents/middleware/subagents.py` | Core subagent specs, tool description, and middleware integration |
| `libs/deepagents/deepagents/middleware/async_subagents.py` | Remote/background subagent handling |
| `libs/cli/deepagents_cli/subagents.py` | CLI loader for filesystem-defined subagents |
| `libs/cli/deepagents_cli/agent.py` | Loads async subagent definitions from config and passes them into CLI agent creation |

## See Also

- [Graph Factory](graph-factory.md)
- [CLI Runtime](cli-runtime.md)
- [Filesystem First Agent Configuration](../concepts/filesystem-first-agent-configuration.md)
- [Remote Subagent and Sandbox Flow](../syntheses/remote-subagent-and-sandbox-flow.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
