# Canvas Voice and Device Control Loop

## Overview

This synthesis explains how OpenClaw's visual, voice, and device-control surfaces combine into one embodied interaction loop.

This synthesis is where the control-plane nature of OpenClaw becomes visible, because no single entity page can show how routing, sessions, plugins, devices, and channels cooperate. The point of this page is to make the handoff between systems explicit so a reader can reason about failures, extension points, and policy boundaries without re-deriving the whole path from raw source.

## Systems Involved

| System | Contribution |
| --- | --- |
| [Device Augmented Agent Architecture](../concepts/device-augmented-agent-architecture.md) | Nodes, canvas, apps, and device execution as agent extensions |
| [Local First Personal Assistant Architecture](../concepts/local-first-personal-assistant-architecture.md) | OpenClaw as a user-owned assistant centered on a local gateway |
| [Canvas and Control UI](../entities/canvas-and-control-ui.md) | Canvas host, A2UI, and web/control-plane presentation surfaces |
| [Node Host and Device Pairing](../entities/node-host-and-device-pairing.md) | Remote device execution, pairing, invocation, and capability policies |
| [Gateway As Control Plane](../concepts/gateway-as-control-plane.md) | Related page in this wiki. |
| [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md) | Related page in this wiki. |

## Interaction Model

Read this interaction model as a boundary-by-boundary path rather than a single function call:
1. A platform client or paired node presents a visual, voice, or device surface.
2. The gateway coordinates identity, events, and capability metadata.
3. Canvas or voice interactions feed the assistant runtime.
4. Resulting actions may target device commands, canvas updates, or spoken responses.
5. The client or node reflects the updated assistant state back to the user.
6. [Canvas and Control UI](../entities/canvas-and-control-ui.md) contributes one stage of the cross-system path described by this synthesis.
7. [Media and Voice Stack](../entities/media-and-voice-stack.md) contributes one stage of the cross-system path described by this synthesis.
8. [Node Host and Device Pairing](../entities/node-host-and-device-pairing.md) contributes one stage of the cross-system path described by this synthesis.
9. [Native Apps and Platform Clients](../entities/native-apps-and-platform-clients.md) contributes one stage of the cross-system path described by this synthesis.

What matters here is where ownership changes hands. One subsystem usually receives input, another owns durable state, another chooses policy or execution strategy, and another surfaces the result back to a user or peer system. If those boundaries are not visible, the runtime looks simpler than it actually is and changes become riskier.

## Key Interfaces

| Interface Or Contract | Why It Matters |
| --- | --- |
| `Canvas HTTP/WebSocket host implementation` | Boundary surface called out by the current page or inferred from the cited sources. |
| `A2UI request handling and live-reload injection` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Shared media handling utilities and payload flows` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Generated media runtime support` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Main node-host process and gateway event loop` | Boundary surface called out by the current page or inferred from the cited sources. |

## Operational Consequences

The synthesis view is usually where debugging stops being guesswork. A symptom that appears in the UI or CLI often originates several layers deeper, especially in repos that separate session state, routing, protocol bridging, plugin activation, and persistence. By following the sequence above, an implementer can decide whether a bug belongs in the initiating surface, the policy layer, the execution layer, or the state-reconciliation layer.

This also explains why change review should span more than one page. A modification that looks local inside an entity page may still alter a synthesis if it changes when state is persisted, which system chooses the active model, how approvals are routed, or where an outbound reply is emitted.

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `src/canvas-host/server.ts` | Canvas HTTP/WebSocket host implementation |
| `src/canvas-host/a2ui.ts` | A2UI request handling and live-reload injection |
| `src/media/` | Shared media handling utilities and payload flows |
| `src/media-generation/` | Generated media runtime support |
| `src/node-host/runner.ts` | Main node-host process and gateway event loop |
| `src/node-host/invoke.ts` | Invocation handling |

## See Also

- [Device Augmented Agent Architecture](../concepts/device-augmented-agent-architecture.md)
- [Local First Personal Assistant Architecture](../concepts/local-first-personal-assistant-architecture.md)
- [Canvas and Control UI](../entities/canvas-and-control-ui.md)
- [Node Host and Device Pairing](../entities/node-host-and-device-pairing.md)
- [Gateway As Control Plane](../concepts/gateway-as-control-plane.md)
- [Inbound Message To Agent Reply Flow](../syntheses/inbound-message-to-agent-reply-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
