# Agent Runtime

## Overview

The agent runtime is the execution core that turns OpenClaw from a messaging gateway into an assistant. It manages prompts, tools, sandbox edges, embedded runners, agent-scoped workspaces, and the Pi-oriented runtime modes referenced throughout the repo and docs.

The agent runtime sits inside a broader platform, so model calls, tools, and compaction are constrained by channel, gateway, and plugin policy.

In practice, this page should be read as a boundary explanation, not just a file list. Agent Runtime owns a specific slice of the OpenClaw runtime, but it only makes sense in relation to neighboring systems such as [Session System](session-system.md), [Skills Platform](skills-platform.md), [Local First Personal Assistant Architecture](../concepts/local-first-personal-assistant-architecture.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `src/agents/agent-scope.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `src/agents/pi-embedded-runner/`, `src/agents/skills/refresh.ts`, `src/agents/subagent-registry.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Agent identity, default-agent, and workspace scoping helpers; Embedded runner lifecycle and run counting; Skill refresh listener registration; Subagent registry initialization |
| Extension or policy points | The main extension or gating surfaces live around `src/agents/sandbox/`. |
| Persistent effects | Sandbox-related runtime behavior |

## Architecture

Agent Runtime is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `src/agents/agent-scope.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Agent Runtime does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `agents/agent-scope.ts` becomes relevant when agent identity, default-agent, and workspace scoping helpers.
2. `pi-embedded-runner/` becomes relevant when embedded runner lifecycle and run counting.
3. `skills/refresh.ts` becomes relevant when skill refresh listener registration.
4. `agents/subagent-registry.ts` becomes relevant when subagent registry initialization.
5. `tools/` becomes relevant when agent-facing tool integrations.
6. `sandbox/` becomes relevant when sandbox-related runtime behavior.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `src/agents/sandbox/`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `src/agents/agent-scope.ts` | Agent identity, default-agent, and workspace scoping helpers |
| `src/agents/pi-embedded-runner/` | Embedded runner lifecycle and run counting |
| `src/agents/skills/refresh.ts` | Skill refresh listener registration |
| `src/agents/subagent-registry.ts` | Subagent registry initialization |
| `src/agents/tools/` | Agent-facing tool integrations |
| `src/agents/sandbox/` | Sandbox-related runtime behavior |

## See Also

- [Session System](session-system.md)
- [Skills Platform](skills-platform.md)
- [Local First Personal Assistant Architecture](../concepts/local-first-personal-assistant-architecture.md)
- [Inbound Message to Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
