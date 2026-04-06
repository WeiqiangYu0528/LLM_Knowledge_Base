# Gateway Control Plane

## Overview

The gateway is the operational center of OpenClaw. It is responsible for loading runtime configuration, authenticating control-plane clients, bootstrapping plugins, hosting sessions and channels, starting cron and auxiliary services, serving the control UI, and coordinating node/device connections. The README explicitly frames this as the real product surface: channels, apps, and agents all orbit the gateway rather than replacing it.

The gateway owns enough runtime state that it is more accurate to call it a control plane than a web server.

In practice, this page should be read as a boundary explanation, not just a file list. Gateway Control Plane owns a specific slice of the OpenClaw runtime, but it only makes sense in relation to neighboring systems such as [CLI and Onboarding](cli-and-onboarding.md), [Plugin Platform](plugin-platform.md), [Gateway as Control Plane](../concepts/gateway-as-control-plane.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

```ts
import { createChannelManager } from "./server-channels.js";
import { buildGatewayCronService } from "./server-cron.js";
import { loadGatewayStartupPlugins } from "./server-plugin-bootstrap.js";
import { createGatewayRuntimeState } from "./server-runtime-state.js";
import { attachGatewayWsHandlers } from "./server-ws-runtime.js";
```

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `src/gateway/server.impl.ts` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `src/gateway/server-methods.ts`, `src/gateway/server-plugin-bootstrap.ts`, `src/gateway/server-channels.ts` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Main composition root for gateway startup and subsystem wiring; Core control-plane method handlers; Startup and deferred plugin loading; Channel manager construction |
| Extension or policy points | The main extension or gating surfaces live around `src/gateway/server-plugin-bootstrap.ts`. |
| Persistent effects | Main composition root for gateway startup and subsystem wiring; Core control-plane method handlers; Startup and deferred plugin loading |

## Architecture

Gateway Control Plane is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `src/gateway/server.impl.ts`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Gateway Control Plane does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `gateway/server.impl.ts` becomes relevant when main composition root for gateway startup and subsystem wiring.
2. `gateway/server-methods.ts` becomes relevant when core control-plane method handlers.
3. `gateway/server-plugin-bootstrap.ts` becomes relevant when startup and deferred plugin loading.
4. `gateway/server-channels.ts` becomes relevant when channel manager construction.
5. `gateway/server-ws-runtime.ts` becomes relevant when websocket attachment and runtime event serving.
6. `gateway/server-cron.ts` becomes relevant when cron service integration into gateway startup.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `src/gateway/server-plugin-bootstrap.ts`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `src/gateway/server.impl.ts` | Main composition root for gateway startup and subsystem wiring |
| `src/gateway/server-methods.ts` | Core control-plane method handlers |
| `src/gateway/server-plugin-bootstrap.ts` | Startup and deferred plugin loading |
| `src/gateway/server-channels.ts` | Channel manager construction |
| `src/gateway/server-ws-runtime.ts` | Websocket attachment and runtime event serving |
| `src/gateway/server-cron.ts` | Cron service integration into gateway startup |
| `src/gateway/mcp-http.ts` | Gateway-level MCP loopback startup |

## See Also

- [CLI and Onboarding](cli-and-onboarding.md)
- [Plugin Platform](plugin-platform.md)
- [Gateway as Control Plane](../concepts/gateway-as-control-plane.md)
- [Onboarding to Live Gateway Flow](../syntheses/onboarding-to-live-gateway-flow.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
