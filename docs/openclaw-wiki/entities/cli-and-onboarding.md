# CLI and Onboarding

## Overview

The CLI is the main operator surface for OpenClaw. It handles onboarding, gateway startup, doctor, updates, channel setup, direct agent messaging, and many maintenance flows. It is not a thin wrapper around one command; it is the interface that brings the gateway, plugins, and config system into a usable local product.

CLI flows in OpenClaw are setup and operations tooling for the gateway, plugins, channels, and device surfaces.

In practice, this page should be read as a boundary explanation, not just a file list. CLI and Onboarding owns a specific slice of the OpenClaw runtime, but it only makes sense in relation to neighboring systems such as [Gateway Control Plane](gateway-control-plane.md), [Plugin Platform](plugin-platform.md), [Onboarding to Live Gateway Flow](../syntheses/onboarding-to-live-gateway-flow.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `src/cli/run-main.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `src/cli/program.ts`, `src/commands/`, `src/wizard/setup.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Top-level CLI bootstrap, env normalization, and command routing; Commander program construction; High-level command implementations such as agent, doctor, models, setup, and status; Guided onboarding/setup workflow |
| Extension or policy points | The main extension or gating surfaces live around the neighboring modules listed in the source map. |
| Persistent effects | state changes and runtime handoffs described in the cited files |

## Architecture

CLI and Onboarding is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `src/cli/run-main.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what CLI and Onboarding does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `cli/run-main.ts` becomes relevant when top-level CLI bootstrap, env normalization, and command routing.
2. `cli/program.ts` becomes relevant when commander program construction.
3. `commands/` becomes relevant when high-level command implementations such as agent, doctor, models, setup, and status.
4. `wizard/setup.ts` becomes relevant when guided onboarding/setup workflow.
5. `README.md` becomes relevant when user-facing install, quick-start, and onboarding guidance.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around the neighboring modules listed in the source map. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `src/cli/run-main.ts` | Top-level CLI bootstrap, env normalization, and command routing |
| `src/cli/program.ts` | Commander program construction |
| `src/commands/` | High-level command implementations such as agent, doctor, models, setup, and status |
| `src/wizard/setup.ts` | Guided onboarding/setup workflow |
| `README.md` | User-facing install, quick-start, and onboarding guidance |

## See Also

- [Gateway Control Plane](gateway-control-plane.md)
- [Plugin Platform](plugin-platform.md)
- [Onboarding to Live Gateway Flow](../syntheses/onboarding-to-live-gateway-flow.md)
- [Local First Personal Assistant Architecture](../concepts/local-first-personal-assistant-architecture.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
