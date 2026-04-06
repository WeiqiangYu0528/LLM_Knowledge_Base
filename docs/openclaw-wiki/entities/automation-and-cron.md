# Automation and Cron

## Overview

Automation in OpenClaw is built around cron-like scheduled jobs and isolated agent execution, not just external task runners. The cron subsystem normalizes schedules, stores jobs, manages timers, executes isolated agent turns, handles delivery, records run logs, and reaps session state when background work finishes.

Scheduled work is modeled as isolated assistant execution, which keeps automation aligned with the rest of the runtime.

In practice, this page should be read as a boundary explanation, not just a file list. Automation and Cron owns a specific slice of the OpenClaw runtime, but it only makes sense in relation to neighboring systems such as [Gateway Control Plane](gateway-control-plane.md), [Session System](session-system.md), [Isolated Agent Automation](../concepts/isolated-agent-automation.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `src/cron/isolated-agent.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `src/cron/isolated-agent/run.ts`, `src/cron/service/jobs.ts`, `src/cron/service/store.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Public isolated-agent execution export; Core isolated-agent run path; Job execution and scheduling logic; Job persistence |
| Extension or policy points | The main extension or gating surfaces live around the neighboring modules listed in the source map. |
| Persistent effects | Public isolated-agent execution export; Core isolated-agent run path; Job execution and scheduling logic |

## Architecture

Automation and Cron is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `src/cron/isolated-agent.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Automation and Cron does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `cron/isolated-agent.ts` becomes relevant when public isolated-agent execution export.
2. `isolated-agent/run.ts` becomes relevant when core isolated-agent run path.
3. `service/jobs.ts` becomes relevant when job execution and scheduling logic.
4. `service/store.ts` becomes relevant when job persistence.
5. `service/timer.ts` becomes relevant when timer management.
6. `cron/run-log.ts` becomes relevant when execution log support.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around the neighboring modules listed in the source map. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `src/cron/isolated-agent.ts` | Public isolated-agent execution export |
| `src/cron/isolated-agent/run.ts` | Core isolated-agent run path |
| `src/cron/service/jobs.ts` | Job execution and scheduling logic |
| `src/cron/service/store.ts` | Job persistence |
| `src/cron/service/timer.ts` | Timer management |
| `src/cron/run-log.ts` | Execution log support |
| `src/cron/session-reaper.ts` | Background session cleanup |

## See Also

- [Gateway Control Plane](gateway-control-plane.md)
- [Session System](session-system.md)
- [Isolated Agent Automation](../concepts/isolated-agent-automation.md)
- [Scheduled Job Delivery Flow](../syntheses/scheduled-job-delivery-flow.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
