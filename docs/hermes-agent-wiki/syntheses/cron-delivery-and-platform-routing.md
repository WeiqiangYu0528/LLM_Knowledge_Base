# Cron Delivery and Platform Routing

## Overview

Cron jobs in Hermes are not just scheduled prompts. They are scheduled prompts that execute in fresh sessions and then hand results off to the same routing and delivery infrastructure used by the messaging gateway. This page explains how that works and why Hermes deliberately keeps cron session state isolated from ordinary chat sessions.

## Systems Involved

- [Cron System](../entities/cron-system.md)
- [Gateway Runtime](../entities/gateway-runtime.md)
- [Messaging Platform Adapters](../entities/messaging-platform-adapters.md)
- [Session Storage](../entities/session-storage.md)

## Interaction Model

The flow is:

1. A due job is selected by the scheduler.
2. The scheduler creates a fresh agent session with no inherited conversation history.
3. The job prompt and any attached skills are prepared.
4. Hermes runs the job through `AIAgent`.
5. The scheduler resolves the delivery target:
   - local output
   - origin chat
   - home channel
   - explicit platform destination
6. If a live gateway adapter is available, Hermes may use it directly for delivery.
7. The output is delivered, wrapped, or suppressed depending on job configuration.
8. The job state is advanced to its next run or completion.

The main ownership handoff is between the scheduler and the gateway delivery layer. Cron owns job timing and run isolation; gateway-style adapters own how a message is actually emitted to a platform.

## Key Interfaces

| Interface | Role |
|------|------|
| `scheduler.tick()` | Due-job execution loop |
| `_resolve_delivery_target(...)` | Turns job config into a concrete route |
| live adapter delivery path | Uses an already-connected gateway adapter when available |
| `SILENT_MARKER` | Lets jobs suppress external delivery while still saving output locally |

The deliberate boundary condition is that cron output is not mirrored into the target chat's ordinary conversation history. Hermes avoids violating message alternation rules or polluting unrelated live chat context with job-execution internals.

## Source Evidence

- `cron/scheduler.py` — scheduler loop, file locking, delivery-target resolution
- `cron/jobs.py` — job storage and lifecycle state
- `gateway/delivery.py` and platform adapters — actual outbound emission behavior
- `website/docs/developer-guide/cron-internals.md` — maintainer explanation of fresh-session isolation and delivery options

## See Also

- [Cron System](../entities/cron-system.md)
- [Gateway Runtime](../entities/gateway-runtime.md)
- [Messaging Platform Adapters](../entities/messaging-platform-adapters.md)
- [Multi-Surface Session Continuity](../concepts/multi-surface-session-continuity.md)
