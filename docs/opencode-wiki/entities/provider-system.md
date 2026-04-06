# Provider System

## Overview

The provider system is OpenCode's provider-agnostic model layer. It integrates many AI SDK backends, chooses model-loading behavior, merges auth/config/environment inputs, and supports special-case logic for providers like OpenAI, GitHub Copilot, Azure, and OpenCode's own hosted provider.

Provider routing is the normalization layer that lets many model vendors fit behind one execution story.

In practice, this page should be read as a boundary explanation, not just a file list. Provider System owns a specific slice of the OpenCode runtime, but it only makes sense in relation to neighboring systems such as [Tool System](tool-system.md), [MCP and ACP Integration](mcp-and-acp-integration.md), [Provider Agnostic Model Routing](../concepts/provider-agnostic-model-routing.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `packages/opencode/src/provider/provider.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `packages/opencode/src/provider/schema.ts`, `packages/opencode/src/provider/models.ts`, `packages/opencode/src/provider/auth.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Provider and model construction logic; Provider/model identifiers and schemas; Model inventory helpers; Provider auth support |
| Extension or policy points | The main extension or gating surfaces live around `packages/opencode/src/provider/provider.ts`, `packages/opencode/src/provider/schema.ts`, `packages/opencode/src/provider/models.ts`, `packages/opencode/src/provider/auth.ts`. |
| Persistent effects | state changes and runtime handoffs described in the cited files |

## Architecture

Provider System is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `packages/opencode/src/provider/provider.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Provider System does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `provider/provider.ts` becomes relevant when provider and model construction logic.
2. `provider/schema.ts` becomes relevant when provider/model identifiers and schemas.
3. `provider/models.ts` becomes relevant when model inventory helpers.
4. `provider/auth.ts` becomes relevant when provider auth support.
5. `provider/transform.ts` becomes relevant when provider transform logic.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `packages/opencode/src/provider/provider.ts`, `packages/opencode/src/provider/schema.ts`, `packages/opencode/src/provider/models.ts`, `packages/opencode/src/provider/auth.ts`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `packages/opencode/src/provider/provider.ts` | Provider and model construction logic |
| `packages/opencode/src/provider/schema.ts` | Provider/model identifiers and schemas |
| `packages/opencode/src/provider/models.ts` | Model inventory helpers |
| `packages/opencode/src/provider/auth.ts` | Provider auth support |
| `packages/opencode/src/provider/transform.ts` | Provider transform logic |

## See Also

- [Tool System](tool-system.md)
- [MCP and ACP Integration](mcp-and-acp-integration.md)
- [Provider Agnostic Model Routing](../concepts/provider-agnostic-model-routing.md)
- [Provider Tool Plugin Interaction Model](../syntheses/provider-tool-plugin-interaction-model.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
