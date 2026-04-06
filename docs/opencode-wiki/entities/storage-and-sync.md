# Storage and Sync

## Overview

The storage and sync subsystem handles persistence, legacy JSON migration, and event-sourced synchronization. It is one of the least visible but most structurally important parts of the OpenCode runtime.

Persistence in OpenCode is eventful and multi-surface: it has to support restore, projection, and sync rather than just saving rows.

In practice, this page should be read as a boundary explanation, not just a file list. Storage and Sync owns a specific slice of the OpenCode runtime, but it only makes sense in relation to neighboring systems such as [Session System](session-system.md), [Control Plane and Workspaces](control-plane-and-workspaces.md), [Context Compaction and Session Recovery](../syntheses/context-compaction-and-session-recovery.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `packages/opencode/src/storage/storage.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `packages/opencode/src/storage/db.ts`, `packages/opencode/src/storage/json-migration.ts`, `packages/opencode/src/sync/index.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Storage interface, migration logic, and JSON persistence helpers; Database access layer; One-time legacy migration logic used at startup; Sync-event registry, replay, and projector flow |
| Extension or policy points | The main extension or gating surfaces live around the neighboring modules listed in the source map. |
| Persistent effects | Storage interface, migration logic, and JSON persistence helpers; Database access layer; Sync-event registry, replay, and projector flow |

## Architecture

Storage and Sync is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `packages/opencode/src/storage/storage.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Storage and Sync does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `storage/storage.ts` becomes relevant when storage interface, migration logic, and JSON persistence helpers.
2. `storage/db.ts` becomes relevant when database access layer.
3. `storage/json-migration.ts` becomes relevant when one-time legacy migration logic used at startup.
4. `sync/index.ts` becomes relevant when sync-event registry, replay, and projector flow.
5. `sync/event.sql.ts` becomes relevant when event persistence tables.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around the neighboring modules listed in the source map. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `packages/opencode/src/storage/storage.ts` | Storage interface, migration logic, and JSON persistence helpers |
| `packages/opencode/src/storage/db.ts` | Database access layer |
| `packages/opencode/src/storage/json-migration.ts` | One-time legacy migration logic used at startup |
| `packages/opencode/src/sync/index.ts` | Sync-event registry, replay, and projector flow |
| `packages/opencode/src/sync/event.sql.ts` | Event persistence tables |

## See Also

- [Session System](session-system.md)
- [Control Plane and Workspaces](control-plane-and-workspaces.md)
- [Context Compaction and Session Recovery](../syntheses/context-compaction-and-session-recovery.md)
- [Request to Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
