# Routing System

## Overview

The routing system decides which agent and session lane should own each inbound interaction. It takes channel, account, peer, guild/team, and role information, applies configured bindings, and emits a resolved `agentId`, `accountId`, `sessionKey`, and route match reason.

Routing determines which agent and transcript own an interaction; mistakes here collapse isolation across channels.

In practice, this page should be read as a boundary explanation, not just a file list. Routing System owns a specific slice of the OpenClaw runtime, but it only makes sense in relation to neighboring systems such as [Session System](session-system.md), [Channel System](channel-system.md), [Multi Channel Session Routing](../concepts/multi-channel-session-routing.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `src/routing/resolve-route.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `src/routing/session-key.ts`, `src/routing/bindings.ts`, `src/routing/account-lookup.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Main route resolution logic; Session key construction and normalization helpers; Binding inventory used during route matching; Account lookup support |
| Extension or policy points | The main extension or gating surfaces live around the neighboring modules listed in the source map. |
| Persistent effects | Session key construction and normalization helpers |

## Architecture

Routing System is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `src/routing/resolve-route.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Routing System does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `routing/resolve-route.ts` becomes relevant when main route resolution logic.
2. `routing/session-key.ts` becomes relevant when session key construction and normalization helpers.
3. `routing/bindings.ts` becomes relevant when binding inventory used during route matching.
4. `routing/account-lookup.ts` becomes relevant when account lookup support.
5. `routing/default-account-warnings.ts` becomes relevant when warnings around implicit/default account routing.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around the neighboring modules listed in the source map. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `src/routing/resolve-route.ts` | Main route resolution logic |
| `src/routing/session-key.ts` | Session key construction and normalization helpers |
| `src/routing/bindings.ts` | Binding inventory used during route matching |
| `src/routing/account-lookup.ts` | Account lookup support |
| `src/routing/default-account-warnings.ts` | Warnings around implicit/default account routing |

## See Also

- [Session System](session-system.md)
- [Channel System](channel-system.md)
- [Multi Channel Session Routing](../concepts/multi-channel-session-routing.md)
- [Channel Binding and Session Identity Flow](../syntheses/channel-binding-and-session-identity-flow.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
