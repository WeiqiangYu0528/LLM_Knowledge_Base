# MCP and ACP Bridges

## Overview

OpenClaw exposes protocol bridges in two directions. The MCP layer turns OpenClaw channels and tools into a Model Context Protocol server surface, while the ACP layer adapts gateway-backed sessions for Agent Client Protocol clients such as editors. Both sit above the gateway rather than replacing it.

Protocol bridges connect OpenClaw to external tools and editors without giving up the gateway's authority over sessions and policy.

In practice, this page should be read as a boundary explanation, not just a file list. MCP and ACP Bridges owns a specific slice of the OpenClaw runtime, but it only makes sense in relation to neighboring systems such as [Gateway Control Plane](gateway-control-plane.md), [Channel System](channel-system.md), [Pluginized Capability Delivery](../concepts/pluginized-capability-delivery.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `src/mcp/channel-server.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `src/mcp/channel-bridge.ts`, `src/mcp/channel-tools.ts`, `src/acp/server.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | MCP server bootstrap around the OpenClaw channel bridge; Gateway-backed channel bridge for MCP; MCP tool registration; ACP server bootstrap and gateway client wiring |
| Extension or policy points | The main extension or gating surfaces live around the neighboring modules listed in the source map. |
| Persistent effects | MCP server bootstrap around the OpenClaw channel bridge; ACP server bootstrap and gateway client wiring; ACP session management |

## Architecture

MCP and ACP Bridges is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `src/mcp/channel-server.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what MCP and ACP Bridges does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `mcp/channel-server.ts` becomes relevant when mCP server bootstrap around the OpenClaw channel bridge.
2. `mcp/channel-bridge.ts` becomes relevant when gateway-backed channel bridge for MCP.
3. `mcp/channel-tools.ts` becomes relevant when mCP tool registration.
4. `acp/server.ts` becomes relevant when aCP server bootstrap and gateway client wiring.
5. `acp/translator.ts` becomes relevant when aCP-to-gateway translation layer.
6. `acp/session.ts` becomes relevant when aCP session management.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around the neighboring modules listed in the source map. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `src/mcp/channel-server.ts` | MCP server bootstrap around the OpenClaw channel bridge |
| `src/mcp/channel-bridge.ts` | Gateway-backed channel bridge for MCP |
| `src/mcp/channel-tools.ts` | MCP tool registration |
| `src/acp/server.ts` | ACP server bootstrap and gateway client wiring |
| `src/acp/translator.ts` | ACP-to-gateway translation layer |
| `src/acp/session.ts` | ACP session management |

## See Also

- [Gateway Control Plane](gateway-control-plane.md)
- [Channel System](channel-system.md)
- [Pluginized Capability Delivery](../concepts/pluginized-capability-delivery.md)
- [Inbound Message to Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
