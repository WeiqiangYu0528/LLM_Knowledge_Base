# Scheduled Job Delivery Flow

## Overview

This synthesis describes how a scheduled OpenClaw job is normalized, executed as an isolated agent turn, and optionally delivered back through user-visible channels.

This synthesis is where the control-plane nature of OpenClaw becomes visible, because no single entity page can show how routing, sessions, plugins, devices, and channels cooperate. The point of this page is to make the handoff between systems explicit so a reader can reason about failures, extension points, and policy boundaries without re-deriving the whole path from raw source.

## Systems Involved

| System | Contribution |
| --- | --- |
| [Isolated Agent Automation](../concepts/isolated-agent-automation.md) | Cron jobs and other background tasks as isolated-agent workloads |
| [Auth and Approval Boundaries](../concepts/auth-and-approval-boundaries.md) | Tokens, passwords, allowlists, approvals, and safety envelopes |
| [Automation and Cron](../entities/automation-and-cron.md) | Scheduled jobs, isolated-agent execution, delivery, and timer service |
| [Inbound Message to Agent Reply Flow](inbound-message-to-agent-reply-flow.md) | End-to-end flow from channel ingress through routing, session, agent, and reply dispatch |
| [Gateway As Control Plane](../concepts/gateway-as-control-plane.md) | Related page in this wiki. |
| [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md) | Related page in this wiki. |

## Interaction Model

Read this interaction model as a boundary-by-boundary path rather than a single function call:
1. A job is stored and normalized by the cron service.
2. The timer service decides when it should fire.
3. An isolated agent execution context is built.
4. The agent runtime runs the turn.
5. Delivery logic decides whether and how to send the result.
6. Run logs and session reaping clean up the background workload.
7. [Automation and Cron](../entities/automation-and-cron.md) contributes one stage of the cross-system path described by this synthesis.
8. [Agent Runtime](../entities/agent-runtime.md) contributes one stage of the cross-system path described by this synthesis.
9. [Session System](../entities/session-system.md) contributes one stage of the cross-system path described by this synthesis.
10. [Gateway Control Plane](../entities/gateway-control-plane.md) contributes one stage of the cross-system path described by this synthesis.

What matters here is where ownership changes hands. One subsystem usually receives input, another owns durable state, another chooses policy or execution strategy, and another surfaces the result back to a user or peer system. If those boundaries are not visible, the runtime looks simpler than it actually is and changes become riskier.

## Key Interfaces

| Interface Or Contract | Why It Matters |
| --- | --- |
| `Public isolated-agent execution export` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Core isolated-agent run path` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Agent identity, default-agent, and workspace scoping helpers` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Embedded runner lifecycle and run counting` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Session ID validation helper` | Boundary surface called out by the current page or inferred from the cited sources. |

## Operational Consequences

The synthesis view is usually where debugging stops being guesswork. A symptom that appears in the UI or CLI often originates several layers deeper, especially in repos that separate session state, routing, protocol bridging, plugin activation, and persistence. By following the sequence above, an implementer can decide whether a bug belongs in the initiating surface, the policy layer, the execution layer, or the state-reconciliation layer.

This also explains why change review should span more than one page. A modification that looks local inside an entity page may still alter a synthesis if it changes when state is persisted, which system chooses the active model, how approvals are routed, or where an outbound reply is emitted.

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

- [Isolated Agent Automation](../concepts/isolated-agent-automation.md)
- [Auth and Approval Boundaries](../concepts/auth-and-approval-boundaries.md)
- [Automation and Cron](../entities/automation-and-cron.md)
- [Inbound Message to Agent Reply Flow](inbound-message-to-agent-reply-flow.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
