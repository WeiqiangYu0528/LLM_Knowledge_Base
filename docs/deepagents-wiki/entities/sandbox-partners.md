# Sandbox Partners

## Overview

The partner packages extend Deep Agents into additional execution environments. Most of them expose sandbox backends for running code remotely; one, QuickJS, adds a lightweight computation middleware rather than a full remote sandbox.

Partner packages matter because remote execution is a product surface, not merely a transport detail.

In practice, this page should be read as a boundary explanation, not just a file list. Sandbox Partners owns a specific slice of the Deep Agents runtime, but it only makes sense in relation to neighboring systems such as [Backend System](backend-system.md), [CLI Runtime](cli-runtime.md), [Local vs Remote Execution](../concepts/local-vs-remote-execution.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `libs/partners/daytona/README.md` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `libs/partners/modal/README.md`, `libs/partners/runloop/README.md`, `libs/partners/quickjs/README.md` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Daytona sandbox package; Modal sandbox package; Runloop sandbox package; QuickJS REPL middleware package |
| Extension or policy points | The main extension or gating surfaces live around `libs/partners/daytona/README.md`, `libs/partners/modal/README.md`, `libs/partners/runloop/README.md`. |
| Persistent effects | Daytona sandbox package; Modal sandbox package; Runloop sandbox package |

## Architecture

Sandbox Partners is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `libs/partners/daytona/README.md`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Sandbox Partners does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `daytona/README.md` becomes relevant when daytona sandbox package.
2. `modal/README.md` becomes relevant when modal sandbox package.
3. `runloop/README.md` becomes relevant when runloop sandbox package.
4. `quickjs/README.md` becomes relevant when quickJS REPL middleware package.
5. `libs/README.md` becomes relevant when monorepo inventory that positions `partners/` as integration packages.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `libs/partners/daytona/README.md`, `libs/partners/modal/README.md`, `libs/partners/runloop/README.md`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `libs/partners/daytona/README.md` | Daytona sandbox package |
| `libs/partners/modal/README.md` | Modal sandbox package |
| `libs/partners/runloop/README.md` | Runloop sandbox package |
| `libs/partners/quickjs/README.md` | QuickJS REPL middleware package |
| `libs/README.md` | Monorepo inventory that positions `partners/` as integration packages |

## See Also

- [Backend System](backend-system.md)
- [CLI Runtime](cli-runtime.md)
- [Local vs Remote Execution](../concepts/local-vs-remote-execution.md)
- [Remote Subagent and Sandbox Flow](../syntheses/remote-subagent-and-sandbox-flow.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
