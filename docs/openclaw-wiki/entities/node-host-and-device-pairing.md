# Node Host and Device Pairing

## Overview

The node host subsystem lets OpenClaw execute commands and device actions through paired clients or remote hosts. It manages gateway connection details, device identity, execution policies, invoke payload handling, plugin-contributed host capabilities, and the paired relationship between gateway and node-like clients.

Paired devices and node hosts extend the assistant into the physical world under gateway-owned policy.

In practice, this page should be read as a boundary explanation, not just a file list. Node Host and Device Pairing owns a specific slice of the OpenClaw runtime, but it only makes sense in relation to neighboring systems such as [Gateway Control Plane](gateway-control-plane.md), [Canvas and Control UI](canvas-and-control-ui.md), [Device Augmented Agent Architecture](../concepts/device-augmented-agent-architecture.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `src/node-host/runner.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `src/node-host/invoke.ts`, `src/node-host/invoke-system-run.ts`, `src/node-host/exec-policy.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Main node-host process and gateway event loop; Invocation handling; System run invocation path; Execution policy enforcement |
| Extension or policy points | The main extension or gating surfaces live around `src/node-host/exec-policy.ts`, `src/node-host/plugin-node-host.ts`. |
| Persistent effects | state changes and runtime handoffs described in the cited files |

## Architecture

Node Host and Device Pairing is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `src/node-host/runner.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Node Host and Device Pairing does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `node-host/runner.ts` becomes relevant when main node-host process and gateway event loop.
2. `node-host/invoke.ts` becomes relevant when invocation handling.
3. `node-host/invoke-system-run.ts` becomes relevant when system run invocation path.
4. `node-host/exec-policy.ts` becomes relevant when execution policy enforcement.
5. `node-host/plugin-node-host.ts` becomes relevant when plugin-contributed node host capabilities.
6. `pairing/` becomes relevant when device pairing-related flows and helpers.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `src/node-host/exec-policy.ts`, `src/node-host/plugin-node-host.ts`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `src/node-host/runner.ts` | Main node-host process and gateway event loop |
| `src/node-host/invoke.ts` | Invocation handling |
| `src/node-host/invoke-system-run.ts` | System run invocation path |
| `src/node-host/exec-policy.ts` | Execution policy enforcement |
| `src/node-host/plugin-node-host.ts` | Plugin-contributed node host capabilities |
| `src/pairing/` | Device pairing-related flows and helpers |

## See Also

- [Gateway Control Plane](gateway-control-plane.md)
- [Canvas and Control UI](canvas-and-control-ui.md)
- [Device Augmented Agent Architecture](../concepts/device-augmented-agent-architecture.md)
- [Canvas Voice and Device Control Loop](../syntheses/canvas-voice-and-device-control-loop.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
