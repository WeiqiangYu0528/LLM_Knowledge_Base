# Session Persistence

## Overview

Session persistence is the CLI's answer to "resume where I left off." It stores thread metadata and checkpointed conversation state in SQLite, surfaces thread listings, and caches derived metadata like message counts and initial prompts for quick UI access.

Thread recovery matters because the CLI is expected to behave like a long-lived coding assistant rather than a stateless command runner.

In practice, this page should be read as a boundary explanation, not just a file list. Session Persistence owns a specific slice of the Deep Agents runtime, but it only makes sense in relation to neighboring systems such as [CLI Textual UI](cli-textual-ui.md), [CLI Runtime](cli-runtime.md), [Context Management and Summarization](../concepts/context-management-and-summarization.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `libs/cli/deepagents_cli/sessions.py` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `libs/deepagents/deepagents/middleware/summarization.py`, `libs/cli/deepagents_cli/_session_stats.py` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | SQLite thread metadata, caches, and helper formatting; Writes offloaded conversation history that complements persisted state; UI-facing session metrics and status rendering |
| Extension or policy points | The main extension or gating surfaces live around the neighboring modules listed in the source map. |
| Persistent effects | SQLite thread metadata, caches, and helper formatting; Writes offloaded conversation history that complements persisted state; UI-facing session metrics and status rendering |

## Architecture

Session Persistence is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `libs/cli/deepagents_cli/sessions.py`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Session Persistence does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `deepagents_cli/sessions.py` becomes relevant when sQLite thread metadata, caches, and helper formatting.
2. `middleware/summarization.py` becomes relevant when writes offloaded conversation history that complements persisted state.
3. `deepagents_cli/_session_stats.py` becomes relevant when uI-facing session metrics and status rendering.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around the neighboring modules listed in the source map. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `libs/cli/deepagents_cli/sessions.py` | SQLite thread metadata, caches, and helper formatting |
| `libs/deepagents/deepagents/middleware/summarization.py` | Writes offloaded conversation history that complements persisted state |
| `libs/cli/deepagents_cli/_session_stats.py` | UI-facing session metrics and status rendering |

## See Also

- [CLI Textual UI](cli-textual-ui.md)
- [CLI Runtime](cli-runtime.md)
- [Context Management and Summarization](../concepts/context-management-and-summarization.md)
- [Interactive Session Lifecycle](../syntheses/interactive-session-lifecycle.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
