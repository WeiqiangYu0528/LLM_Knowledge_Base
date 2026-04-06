# Plugin Platform

## Overview

The plugin platform is the main extensibility substrate in OpenClaw. It discovers plugins, validates manifests, resolves activation rules, constructs runtime APIs, manages install/uninstall state, exposes interactive and command surfaces, and maintains the active registry of channels, providers, tools, hooks, and memory/runtime supplements.

Plugins are the delivery mechanism for much of OpenClaw's capability surface, so loader and runtime rules shape the whole platform.

In practice, this page should be read as a boundary explanation, not just a file list. Plugin Platform owns a specific slice of the OpenClaw runtime, but it only makes sense in relation to neighboring systems such as [Provider and Model System](provider-and-model-system.md), [Channel Plugin Adapters](channel-plugin-adapters.md), [Pluginized Capability Delivery](../concepts/pluginized-capability-delivery.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `src/plugins/loader.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `src/plugins/runtime.ts`, `src/plugins/manifest-registry.ts`, `src/plugins/install.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Central discovery, load, validation, and activation path; Active registry lifecycle and runtime state; Manifest inventory loading; Plugin installation flow |
| Extension or policy points | The main extension or gating surfaces live around `src/plugins/loader.ts`, `src/plugins/runtime.ts`, `src/plugins/manifest-registry.ts`, `src/plugins/install.ts`. |
| Persistent effects | state changes and runtime handoffs described in the cited files |

## Architecture

Plugin Platform is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `src/plugins/loader.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Plugin Platform does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `plugins/loader.ts` becomes relevant when central discovery, load, validation, and activation path.
2. `plugins/runtime.ts` becomes relevant when active registry lifecycle and runtime state.
3. `plugins/manifest-registry.ts` becomes relevant when manifest inventory loading.
4. `plugins/install.ts` becomes relevant when plugin installation flow.
5. `plugins/uninstall.ts` becomes relevant when plugin removal flow.
6. `plugins/api-builder.ts` becomes relevant when plugin-facing runtime API construction.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `src/plugins/loader.ts`, `src/plugins/runtime.ts`, `src/plugins/manifest-registry.ts`, `src/plugins/install.ts`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `src/plugins/loader.ts` | Central discovery, load, validation, and activation path |
| `src/plugins/runtime.ts` | Active registry lifecycle and runtime state |
| `src/plugins/manifest-registry.ts` | Manifest inventory loading |
| `src/plugins/install.ts` | Plugin installation flow |
| `src/plugins/uninstall.ts` | Plugin removal flow |
| `src/plugins/api-builder.ts` | Plugin-facing runtime API construction |
| `src/plugins/schema-validator.ts` | Manifest/config schema validation |

## See Also

- [Provider and Model System](provider-and-model-system.md)
- [Channel Plugin Adapters](channel-plugin-adapters.md)
- [Pluginized Capability Delivery](../concepts/pluginized-capability-delivery.md)
- [Extension to Runtime Capability Flow](../syntheses/extension-to-runtime-capability-flow.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
