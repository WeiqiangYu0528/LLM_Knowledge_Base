# Desktop and Remote Clients

## Overview

Beyond the main CLI and app surfaces, OpenCode ships or supports several additional clients and shells: desktop packages, Slack integration, a VS Code SDK surface, and related packaging surfaces. These packages make the repo a product platform rather than only a terminal app.

Desktop and remote clients show how far the core runtime can be reused without changing its internal execution model.

In practice, this page should be read as a boundary explanation, not just a file list. Desktop and Remote Clients owns a specific slice of the OpenCode runtime, but it only makes sense in relation to neighboring systems such as [UI Client Surfaces](ui-client-surfaces.md), [Control Plane and Workspaces](control-plane-and-workspaces.md), [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `packages/desktop/package.json and README.md` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `packages/desktop-electron/package.json and README.md`, `packages/slack/package.json and README.md`, `packages/sdk/js/` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Tauri desktop shell; Electron desktop shell; Slack integration package; JavaScript SDK package |
| Extension or policy points | The main extension or gating surfaces live around the neighboring modules listed in the source map. |
| Persistent effects | Tauri desktop shell; Electron desktop shell |

## Architecture

Desktop and Remote Clients is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `packages/desktop/package.json and README.md`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Desktop and Remote Clients does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `desktop/package.json and README.md` becomes relevant when tauri desktop shell.
2. `desktop-electron/package.json and README.md` becomes relevant when electron desktop shell.
3. `slack/package.json and README.md` becomes relevant when slack integration package.
4. `js/` becomes relevant when javaScript SDK package.
5. `vscode/README.md` becomes relevant when vS Code SDK/client surface.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around the neighboring modules listed in the source map. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `packages/desktop/package.json and README.md` | Tauri desktop shell |
| `packages/desktop-electron/package.json and README.md` | Electron desktop shell |
| `packages/slack/package.json and README.md` | Slack integration package |
| `packages/sdk/js/` | JavaScript SDK package |
| `sdks/vscode/README.md` | VS Code SDK/client surface |

## See Also

- [UI Client Surfaces](ui-client-surfaces.md)
- [Control Plane and Workspaces](control-plane-and-workspaces.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Multi Client Product Architecture](../syntheses/multi-client-product-architecture.md)
- [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
