# Isolated Agent Automation

## Overview

OpenClaw's automation system is built around isolated agent turns. Scheduled and background work is treated as agent execution with explicit isolation, delivery, and cleanup semantics instead of as unrelated cron shell jobs.

This concept is a platform invariant. It explains why the gateway, routing, plugins, channels, and clients can evolve independently without collapsing into contradictory behavior. Isolated Agent Automation is therefore best read as a mechanism with operational consequences, not merely a label for related features.

## Why It Exists

This concept exists because the OpenClaw codebase repeatedly needs a stable way to coordinate behavior across multiple entities without turning those entities into one monolith.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenClaw, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

## Mechanism

A useful way to read this mechanism is as an ordered path through the participating subsystems:
1. Normalize and store a cron job.
2. Fire the timer and construct the job execution context.
3. Run an isolated agent turn with the necessary route/session state.
4. Deliver the result through the normal reply/delivery system when appropriate.
5. Record the run and reap session artifacts as needed.

The steps above are the operational skeleton. The exact file names vary by subsystem, but the concept remains stable because each participating entity contributes one predictable part of the chain. That is why the same concept can show up in SDK code, CLI wiring, plugin activation, channel routing, or persistence without becoming ambiguous.

## Invariants and Operational Implications

The most important invariant is that the linked entities are allowed to change implementation detail without changing the high-level guarantee described here. When a change breaks that guarantee, the failure usually appears at subsystem boundaries first: a summary no longer compacts correctly, a session route stops being stable, a skill path is not loaded consistently, or a permission rule is evaluated in the wrong layer.

Operationally, this means debugging should follow the mechanism rather than a UI symptom. Start where the concept is introduced, then inspect the next boundary where data, policy, or control is handed off. The source evidence table below is organized to support exactly that style of investigation.

## Involved Entities

| Entity | Role In This Concept |
| --- | --- |
| [Gateway as Control Plane](gateway-as-control-plane.md) | Why the gateway owns sessions, channels, tools, config, and UI surfaces |
| [Auth and Approval Boundaries](auth-and-approval-boundaries.md) | Tokens, passwords, allowlists, approvals, and safety envelopes |
| [Automation and Cron](../entities/automation-and-cron.md) | Scheduled jobs, isolated-agent execution, delivery, and timer service |
| [Scheduled Job Delivery Flow](../syntheses/scheduled-job-delivery-flow.md) | How cron normalization, isolated execution, delivery, and cleanup compose |
| [Gateway As Control Plane](../concepts/gateway-as-control-plane.md) | Related page in this wiki. |
| [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md) | Related page in this wiki. |

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `src/cron/isolated-agent.ts` | Public isolated-agent execution export |
| `src/cron/isolated-agent/run.ts` | Core isolated-agent run path |
| `src/agents/agent-scope.ts` | Agent identity, default-agent, and workspace scoping helpers |
| `src/agents/pi-embedded-runner/` | Embedded runner lifecycle and run counting |
| `src/sessions/session-id.ts` | Session ID validation helper |
| `src/sessions/session-lifecycle-events.ts` | Lifecycle event contracts for session creation and teardown |

## See Also

- [Gateway as Control Plane](gateway-as-control-plane.md)
- [Auth and Approval Boundaries](auth-and-approval-boundaries.md)
- [Automation and Cron](../entities/automation-and-cron.md)
- [Scheduled Job Delivery Flow](../syntheses/scheduled-job-delivery-flow.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
