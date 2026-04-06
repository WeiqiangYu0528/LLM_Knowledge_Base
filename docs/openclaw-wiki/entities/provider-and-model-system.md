# Provider and Model System

## Overview

The provider and model system manages how OpenClaw discovers model backends, authenticates with them, selects primary models, and falls back across multiple auth/provider choices. This is both a plugin problem and a product problem, because providers are extension-delivered but model choice shapes the core assistant experience.

Provider selection is mediated through plugins, auth choices, allowlists, and failover rules rather than hardcoded model calls.

In practice, this page should be read as a boundary explanation, not just a file list. Provider and Model System owns a specific slice of the OpenClaw runtime, but it only makes sense in relation to neighboring systems such as [Plugin Platform](plugin-platform.md), [CLI and Onboarding](cli-and-onboarding.md), [Pluginized Capability Delivery](../concepts/pluginized-capability-delivery.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `src/plugins/provider-catalog.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `src/plugins/providers.runtime.ts`, `src/plugins/provider-auth-choice.ts`, `src/plugins/provider-model-primary.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Provider inventory and catalog behavior; Runtime-facing provider assembly; Provider auth decision logic; Primary model selection helpers |
| Extension or policy points | The main extension or gating surfaces live around `src/plugins/provider-catalog.ts`, `src/plugins/providers.runtime.ts`, `src/plugins/provider-auth-choice.ts`, `src/plugins/provider-model-primary.ts`. |
| Persistent effects | state changes and runtime handoffs described in the cited files |

## Architecture

Provider and Model System is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `src/plugins/provider-catalog.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Provider and Model System does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `plugins/provider-catalog.ts` becomes relevant when provider inventory and catalog behavior.
2. `plugins/providers.runtime.ts` becomes relevant when runtime-facing provider assembly.
3. `plugins/provider-auth-choice.ts` becomes relevant when provider auth decision logic.
4. `plugins/provider-model-primary.ts` becomes relevant when primary model selection helpers.
5. `plugins/provider-model-allowlist.ts` becomes relevant when model allowlist/compatibility logic.
6. `plugins/provider-openai-codex-oauth.ts` becomes relevant when example provider-specific OAuth integration.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `src/plugins/provider-catalog.ts`, `src/plugins/providers.runtime.ts`, `src/plugins/provider-auth-choice.ts`, `src/plugins/provider-model-primary.ts`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `src/plugins/provider-catalog.ts` | Provider inventory and catalog behavior |
| `src/plugins/providers.runtime.ts` | Runtime-facing provider assembly |
| `src/plugins/provider-auth-choice.ts` | Provider auth decision logic |
| `src/plugins/provider-model-primary.ts` | Primary model selection helpers |
| `src/plugins/provider-model-allowlist.ts` | Model allowlist/compatibility logic |
| `src/plugins/provider-openai-codex-oauth.ts` | Example provider-specific OAuth integration |
| `extensions/` | Concrete provider plugin packages such as OpenAI, Anthropic, Ollama, and OpenRouter |

## See Also

- [Plugin Platform](plugin-platform.md)
- [CLI and Onboarding](cli-and-onboarding.md)
- [Pluginized Capability Delivery](../concepts/pluginized-capability-delivery.md)
- [Extension to Runtime Capability Flow](../syntheses/extension-to-runtime-capability-flow.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
