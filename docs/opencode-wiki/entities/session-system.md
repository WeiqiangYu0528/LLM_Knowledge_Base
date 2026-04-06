# Session System

## Overview

The session system is the heart of OpenCode's agent runtime. It stores session metadata, titles, permissions, summaries, share state, and revert data, while also coordinating prompts, messages, summaries, compaction, retry behavior, and todo/status tracking.

Sessions in OpenCode are the durable center of the runtime, where prompts, summaries, messages, and retries all converge.

In practice, this page should be read as a boundary explanation, not just a file list. Session System owns a specific slice of the OpenCode runtime, but it only makes sense in relation to neighboring systems such as [Tool System](tool-system.md), [Provider System](provider-system.md), [Context Compaction and Session Recovery](../syntheses/context-compaction-and-session-recovery.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

```ts
Session.Info
Session.GlobalInfo
Session.Event.Created
Session.Event.Updated
Session.Event.Deleted
```

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `packages/opencode/src/session/index.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `packages/opencode/src/session/prompt.ts`, `packages/opencode/src/session/prompt/default.txt, plan.txt, build-switch.txt, anthropic.txt, gpt.txt`, `packages/opencode/src/session/compaction.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Session records, events, plan-file path logic, and usage aggregation; Prompt selection logic; Prompt variants for modes and model families; Context compaction logic |
| Extension or policy points | The main extension or gating surfaces live around the neighboring modules listed in the source map. |
| Persistent effects | Session records, events, plan-file path logic, and usage aggregation; Prompt selection logic; Prompt variants for modes and model families |

## Architecture

Session System is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `packages/opencode/src/session/index.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Session System does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `session/index.ts` becomes relevant when session records, events, plan-file path logic, and usage aggregation.
2. `session/prompt.ts` becomes relevant when prompt selection logic.
3. `prompt/default.txt, plan.txt, build-switch.txt, anthropic.txt, gpt.txt` becomes relevant when prompt variants for modes and model families.
4. `session/compaction.ts` becomes relevant when context compaction logic.
5. `session/summary.ts` becomes relevant when summary generation and maintenance.
6. `session/retry.ts` becomes relevant when retry behavior.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around the neighboring modules listed in the source map. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `packages/opencode/src/session/index.ts` | Session records, events, plan-file path logic, and usage aggregation |
| `packages/opencode/src/session/prompt.ts` | Prompt selection logic |
| `packages/opencode/src/session/prompt/default.txt, plan.txt, build-switch.txt, anthropic.txt, gpt.txt` | Prompt variants for modes and model families |
| `packages/opencode/src/session/compaction.ts` | Context compaction logic |
| `packages/opencode/src/session/summary.ts` | Summary generation and maintenance |
| `packages/opencode/src/session/retry.ts` | Retry behavior |
| `packages/opencode/src/session/revert.ts` | Revert operations |
| `packages/opencode/src/session/message.ts and message-v2.ts` | Message representation |

## See Also

- [Tool System](tool-system.md)
- [Provider System](provider-system.md)
- [Context Compaction and Session Recovery](../syntheses/context-compaction-and-session-recovery.md)
- [Tool and Agent Composition](../concepts/tool-and-agent-composition.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
