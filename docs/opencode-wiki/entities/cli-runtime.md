# CLI Runtime

## Overview

The CLI runtime is the primary entry point into OpenCode. It owns command parsing, logging initialization, one-time database migration, environment markers, and the large command surface that exposes account, providers, agent selection, serving, TUI attach/thread flows, ACP, MCP, import/export, and debugging.

The CLI is where the batteries-included SDK becomes a day-to-day product: config, approvals, sessions, and remote tooling all converge here.

In practice, this page should be read as a boundary explanation, not just a file list. CLI Runtime owns a specific slice of the OpenCode runtime, but it only makes sense in relation to neighboring systems such as [Session System](session-system.md), [Project and Instance System](project-and-instance-system.md), [Request to Session Execution Flow](../syntheses/request-to-session-execution-flow.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `packages/opencode/src/index.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `packages/opencode/src/cli/bootstrap.ts`, `packages/opencode/src/cli/cmd/`, `opencode/package.json` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Root command entry point and startup/migration flow; Project-scoped bootstrap helper; Command implementations across serve, run, ACP, MCP, providers, TUI, and debugging; Workspace scripts and runtime command catalog |
| Extension or policy points | The main extension or gating surfaces live around `packages/opencode/src/cli/cmd/`. |
| Persistent effects | state changes and runtime handoffs described in the cited files |

## Architecture

CLI Runtime is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `packages/opencode/src/index.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what CLI Runtime does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `src/index.ts` becomes relevant when root command entry point and startup/migration flow.
2. `cli/bootstrap.ts` becomes relevant when project-scoped bootstrap helper.
3. `cmd/` becomes relevant when command implementations across serve, run, ACP, MCP, providers, TUI, and debugging.
4. `opencode/package.json` becomes relevant when workspace scripts and runtime command catalog.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `packages/opencode/src/cli/cmd/`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `packages/opencode/src/index.ts` | Root command entry point and startup/migration flow |
| `packages/opencode/src/cli/bootstrap.ts` | Project-scoped bootstrap helper |
| `packages/opencode/src/cli/cmd/` | Command implementations across serve, run, ACP, MCP, providers, TUI, and debugging |
| `opencode/package.json` | Workspace scripts and runtime command catalog |

## See Also

- [Session System](session-system.md)
- [Project and Instance System](project-and-instance-system.md)
- [Request to Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
