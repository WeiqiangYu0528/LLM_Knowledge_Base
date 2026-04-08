# Cron System

## Overview

The cron subsystem gives Hermes a scheduled automation surface on top of the main agent runtime. Jobs can run once, on intervals, or on cron expressions; they execute in fresh sessions; and they can deliver results to local output or supported messaging platforms. The key point is that cron is not a separate automation engine with its own agent. It is a scheduler that instantiates the same Hermes runtime under stricter constraints.

## Key Types / Key Concepts

| Type or file | Role |
|------|------|
| `cron/jobs.py` | Job model, storage, and state transitions |
| `cron/scheduler.py` | Tick loop, locking, and due-job execution |
| `tools/cronjob_tools.py` | Model-facing tool used to create and manage cron jobs |
| `hermes_cli/cron.py` | Direct CLI management surface |
| delivery target resolution | Logic that maps `origin`, `local`, or explicit platform targets into concrete destinations |

## Architecture

Cron is composed of three layers:

1. persistent job definitions in `jobs.json`
2. scheduler logic that finds due jobs and executes them
3. delivery logic that sends outputs to the right destination

The gateway can host the scheduler as part of its maintenance cycle, but cron is still an independent subsystem because it owns job state, repeat counters, locking, and delivery semantics.

## Runtime Behavior

### Scheduling and execution

The scheduler supports several schedule shapes: relative delays, intervals, cron expressions, and ISO timestamps. On each tick it:

1. acquires a file-based lock
2. loads jobs
3. filters due scheduled jobs
4. marks them running
5. creates fresh agent sessions
6. optionally injects skill context
7. runs the prompt
8. delivers the result
9. advances or completes the schedule

### Isolation and recursion guard

Cron sessions are intentionally fresh and isolated:

- no prior conversation history
- no clarifying questions expected
- `cronjob` toolset disabled inside cron-run sessions

That last choice prevents recursive job creation or self-mutating schedule loops from inside scheduled work.

### Delivery semantics

Cron jobs can deliver to:

- local output
- the origin chat
- a configured home channel
- an explicit platform target such as `telegram:<chat_id>`

Those deliveries are not mirrored into ordinary gateway conversation history. Hermes keeps cron sessions separate so that long-lived chat conversations do not become structurally invalid.

## Source Files

| File | Purpose |
|------|---------|
| `cron/jobs.py` | Job definitions and atomic storage |
| `cron/scheduler.py` | Tick loop, lock file, delivery-target resolution, fresh-session execution |
| `tools/cronjob_tools.py` | Model-facing cron-management tool |
| `hermes_cli/cron.py` | CLI management commands |
| `website/docs/developer-guide/cron-internals.md` | Maintainer guide to schedules, lifecycle, and delivery |

## See Also

- [Gateway Runtime](gateway-runtime.md)
- [CLI Runtime](cli-runtime.md)
- [Cron Delivery and Platform Routing](../syntheses/cron-delivery-and-platform-routing.md)
- [Isolated Agent Automation](../../openclaw-wiki/concepts/isolated-agent-automation.md)
