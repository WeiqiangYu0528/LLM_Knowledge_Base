# Skills Platform

## Overview

OpenClaw ships with a skills platform that supports bundled, managed, and workspace skills. Skills are part of onboarding, tool UX, and agent runtime behavior rather than an isolated plugin marketplace. The repo contains both runtime skill code under `src/agents/skills/` and distributable skill content under top-level `skills/`.

Skills in OpenClaw are installable, managed, and workspace-aware, which makes them part of the platform governance story.

In practice, this page should be read as a boundary explanation, not just a file list. Skills Platform owns a specific slice of the OpenClaw runtime, but it only makes sense in relation to neighboring systems such as [Agent Runtime](agent-runtime.md), [Plugin Platform](plugin-platform.md), [Pluginized Capability Delivery](../concepts/pluginized-capability-delivery.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `src/agents/skills/refresh.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `skills/`, `README.md`, `src/gateway/server.impl.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Skill refresh listener support; Bundled skill content shipped with OpenClaw; Product-level framing for onboarding plus bundled/managed/workspace skills; Gateway-side remote skill cache priming and registry wiring |
| Extension or policy points | The main extension or gating surfaces live around `src/gateway/server.impl.ts`. |
| Persistent effects | Gateway-side remote skill cache priming and registry wiring |

## Architecture

Skills Platform is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `src/agents/skills/refresh.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Skills Platform does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `skills/refresh.ts` becomes relevant when skill refresh listener support.
2. `skills/` becomes relevant when bundled skill content shipped with OpenClaw.
3. `README.md` becomes relevant when product-level framing for onboarding plus bundled/managed/workspace skills.
4. `gateway/server.impl.ts` becomes relevant when gateway-side remote skill cache priming and registry wiring.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `src/gateway/server.impl.ts`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `src/agents/skills/refresh.ts` | Skill refresh listener support |
| `skills/` | Bundled skill content shipped with OpenClaw |
| `README.md` | Product-level framing for onboarding plus bundled/managed/workspace skills |
| `src/gateway/server.impl.ts` | Gateway-side remote skill cache priming and registry wiring |

## See Also

- [Agent Runtime](agent-runtime.md)
- [Plugin Platform](plugin-platform.md)
- [Pluginized Capability Delivery](../concepts/pluginized-capability-delivery.md)
- [Extension to Runtime Capability Flow](../syntheses/extension-to-runtime-capability-flow.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
