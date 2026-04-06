# LSP and Code Intelligence

## Overview

OpenCode has explicit language-server support rather than treating code intelligence as incidental. This powers LSP-aware tooling and helps distinguish OpenCode from simpler prompt-only terminal agents.

LSP features are integrated as runtime services and tools, not as a separate IDE-only subsystem.

In practice, this page should be read as a boundary explanation, not just a file list. LSP and Code Intelligence owns a specific slice of the OpenCode runtime, but it only makes sense in relation to neighboring systems such as [Tool System](tool-system.md), [Project and Instance System](project-and-instance-system.md), [Tool and Agent Composition](../concepts/tool-and-agent-composition.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `packages/opencode/src/lsp/index.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `packages/opencode/src/lsp/server.ts`, `packages/opencode/src/lsp/client.ts`, `packages/opencode/src/lsp/launch.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | LSP entry point; LSP server interactions; LSP client logic; Launch/setup flow |
| Extension or policy points | The main extension or gating surfaces live around the neighboring modules listed in the source map. |
| Persistent effects | LSP server interactions |

## Architecture

LSP and Code Intelligence is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `packages/opencode/src/lsp/index.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what LSP and Code Intelligence does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `lsp/index.ts` becomes relevant when lSP entry point.
2. `lsp/server.ts` becomes relevant when lSP server interactions.
3. `lsp/client.ts` becomes relevant when lSP client logic.
4. `lsp/launch.ts` becomes relevant when launch/setup flow.
5. `tool/lsp.ts` becomes relevant when lSP-backed tool implementation.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around the neighboring modules listed in the source map. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `packages/opencode/src/lsp/index.ts` | LSP entry point |
| `packages/opencode/src/lsp/server.ts` | LSP server interactions |
| `packages/opencode/src/lsp/client.ts` | LSP client logic |
| `packages/opencode/src/lsp/launch.ts` | Launch/setup flow |
| `packages/opencode/src/tool/lsp.ts` | LSP-backed tool implementation |

## See Also

- [Tool System](tool-system.md)
- [Project and Instance System](project-and-instance-system.md)
- [Tool and Agent Composition](../concepts/tool-and-agent-composition.md)
- [Request to Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
