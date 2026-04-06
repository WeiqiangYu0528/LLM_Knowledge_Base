# Backend System

## Overview

The backend system abstracts both file storage and, when supported, command execution. This is the mechanism that lets the same agent shape run over in-memory state, the local filesystem, a shell-enabled local environment, or routed combinations of backends.

Backend code in Deep Agents is not just storage plumbing; it determines how the agent sees files, shell access, and remote execution targets.

In practice, this page should be read as a boundary explanation, not just a file list. Backend System owns a specific slice of the Deep Agents runtime, but it only makes sense in relation to neighboring systems such as [Graph Factory](graph-factory.md), [CLI Runtime](cli-runtime.md), [Local vs Remote Execution](../concepts/local-vs-remote-execution.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `libs/deepagents/deepagents/backends/protocol.py` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `libs/deepagents/deepagents/backends/filesystem.py`, `libs/deepagents/deepagents/backends/local_shell.py`, `libs/deepagents/deepagents/backends/composite.py` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Backend and execution protocol definitions plus result types; Concrete filesystem backend with optional virtual path mode; Filesystem backend that also executes unrestricted shell commands; Route-based backend composition by path prefix |
| Extension or policy points | The main extension or gating surfaces live around the neighboring modules listed in the source map. |
| Persistent effects | Filesystem backend that also executes unrestricted shell commands |

## Architecture

Backend System is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `libs/deepagents/deepagents/backends/protocol.py`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Backend System does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `backends/protocol.py` becomes relevant when backend and execution protocol definitions plus result types.
2. `backends/filesystem.py` becomes relevant when concrete filesystem backend with optional virtual path mode.
3. `backends/local_shell.py` becomes relevant when filesystem backend that also executes unrestricted shell commands.
4. `backends/composite.py` becomes relevant when route-based backend composition by path prefix.
5. `backends/__init__.py` becomes relevant when public backend export surface.
6. `middleware/filesystem.py` becomes relevant when tool layer that consumes backend capabilities.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around the neighboring modules listed in the source map. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `libs/deepagents/deepagents/backends/protocol.py` | Backend and execution protocol definitions plus result types |
| `libs/deepagents/deepagents/backends/filesystem.py` | Concrete filesystem backend with optional virtual path mode |
| `libs/deepagents/deepagents/backends/local_shell.py` | Filesystem backend that also executes unrestricted shell commands |
| `libs/deepagents/deepagents/backends/composite.py` | Route-based backend composition by path prefix |
| `libs/deepagents/deepagents/backends/__init__.py` | Public backend export surface |
| `libs/deepagents/deepagents/middleware/filesystem.py` | Tool layer that consumes backend capabilities |

## See Also

- [Graph Factory](graph-factory.md)
- [CLI Runtime](cli-runtime.md)
- [Local vs Remote Execution](../concepts/local-vs-remote-execution.md)
- [Remote Subagent and Sandbox Flow](../syntheses/remote-subagent-and-sandbox-flow.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
