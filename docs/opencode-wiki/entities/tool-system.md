# Tool System

## Overview

The tool system defines OpenCode's actionable surface: bash, file read/write/edit, grep/glob, task, todo, skill, web fetch/search, code search, LSP, patch application, question prompts, and experimental batch/plan tooling. It is also where plugin-defined tools are merged into the built-in catalog.

The tool layer turns a model invocation into a coding agent by defining what external actions exist and how they are described.

In practice, this page should be read as a boundary explanation, not just a file list. Tool System owns a specific slice of the OpenCode runtime, but it only makes sense in relation to neighboring systems such as [Provider System](provider-system.md), [Plugin System](plugin-system.md), [Tool and Agent Composition](../concepts/tool-and-agent-composition.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `packages/opencode/src/tool/registry.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `packages/opencode/src/tool/registry.ts, tool.ts, bash.ts, read.ts, write.ts, task.ts`, `packages/opencode/src/tool/bash.txt, read.txt, write.txt, task.txt, plan-enter.txt, plan-exit.txt`, `packages/opencode/src/plugin/index.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Dynamic tool assembly and filtering; Built-in tool implementations; Tool descriptions/prompt snippets; Provides plugin-defined tools to the registry |
| Extension or policy points | The main extension or gating surfaces live around `packages/opencode/src/plugin/index.ts`. |
| Persistent effects | state changes and runtime handoffs described in the cited files |

## Architecture

Tool System is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `packages/opencode/src/tool/registry.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Tool System does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `tool/registry.ts` becomes relevant when dynamic tool assembly and filtering.
2. `tool/registry.ts, tool.ts, bash.ts, read.ts, write.ts, task.ts` becomes relevant when built-in tool implementations.
3. `tool/bash.txt, read.txt, write.txt, task.txt, plan-enter.txt, plan-exit.txt` becomes relevant when tool descriptions/prompt snippets.
4. `plugin/index.ts` becomes relevant when provides plugin-defined tools to the registry.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `packages/opencode/src/plugin/index.ts`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `packages/opencode/src/tool/registry.ts` | Dynamic tool assembly and filtering |
| `packages/opencode/src/tool/registry.ts, tool.ts, bash.ts, read.ts, write.ts, task.ts` | Built-in tool implementations |
| `packages/opencode/src/tool/bash.txt, read.txt, write.txt, task.txt, plan-enter.txt, plan-exit.txt` | Tool descriptions/prompt snippets |
| `packages/opencode/src/plugin/index.ts` | Provides plugin-defined tools to the registry |

## See Also

- [Provider System](provider-system.md)
- [Plugin System](plugin-system.md)
- [Tool and Agent Composition](../concepts/tool-and-agent-composition.md)
- [Provider Tool Plugin Interaction Model](../syntheses/provider-tool-plugin-interaction-model.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
