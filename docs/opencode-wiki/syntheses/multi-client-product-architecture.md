# Multi Client Product Architecture

## Overview

This synthesis documents OpenCode as a product family rather than a single application. The same core runtime supports multiple user-facing surfaces with different strengths and packaging constraints.

This synthesis matters because OpenCode is multi-surface; the runtime only makes sense when the handoff between client, session, provider, tool, and persistence layers is explicit. The point of this page is to make the handoff between systems explicit so a reader can reason about failures, extension points, and policy boundaries without re-deriving the whole path from raw source.

## Systems Involved

| System | Contribution |
| --- | --- |
| [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md) | OpenCode core service with multiple clients layered on top |
| [UI Client Surfaces](../entities/ui-client-surfaces.md) | Shared app/UI/web layers above the core runtime |
| [Desktop and Remote Clients](../entities/desktop-and-remote-clients.md) | Desktop shells, Slack, VS Code SDK, and other client surfaces |
| [Workspace Routing and Remote Control](workspace-routing-and-remote-control.md) | Local vs remote workspaces, adaptors, and forwarded requests |
| [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md) | Related page in this wiki. |
| [Architecture Overview](../summaries/architecture-overview.md) | High-level orientation to the OpenCode monorepo, package boundaries, and runtime model |

## Interaction Model

Read this interaction model as a boundary-by-boundary path rather than a single function call:
1. [CLI Runtime](../entities/cli-runtime.md) contributes one stage of the cross-system path described by this synthesis.
2. [UI Client Surfaces](../entities/ui-client-surfaces.md) contributes one stage of the cross-system path described by this synthesis.
3. [Desktop and Remote Clients](../entities/desktop-and-remote-clients.md) contributes one stage of the cross-system path described by this synthesis.
4. [Server API](../entities/server-api.md) contributes one stage of the cross-system path described by this synthesis.

What matters here is where ownership changes hands. One subsystem usually receives input, another owns durable state, another chooses policy or execution strategy, and another surfaces the result back to a user or peer system. If those boundaries are not visible, the runtime looks simpler than it actually is and changes become riskier.

## Key Interfaces

| Interface Or Contract | Why It Matters |
| --- | --- |
| `Root command entry point and startup/migration flow` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Project-scoped bootstrap helper` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Main app client package and scripts` | Boundary surface called out by the current page or inferred from the cited sources. |
| `App usage and e2e notes` | Boundary surface called out by the current page or inferred from the cited sources. |
| `and `README.md`: Tauri desktop shell` | Boundary surface called out by the current page or inferred from the cited sources. |

## Operational Consequences

The synthesis view is usually where debugging stops being guesswork. A symptom that appears in the UI or CLI often originates several layers deeper, especially in repos that separate session state, routing, protocol bridging, plugin activation, and persistence. By following the sequence above, an implementer can decide whether a bug belongs in the initiating surface, the policy layer, the execution layer, or the state-reconciliation layer.

This also explains why change review should span more than one page. A modification that looks local inside an entity page may still alter a synthesis if it changes when state is persisted, which system chooses the active model, how approvals are routed, or where an outbound reply is emitted.

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `packages/opencode/src/index.ts` | Root command entry point and startup/migration flow |
| `packages/opencode/src/cli/bootstrap.ts` | Project-scoped bootstrap helper |
| `packages/app/package.json` | Main app client package and scripts |
| `packages/app/README.md` | App usage and e2e notes |
| `packages/desktop/package.json` | and `README.md`: Tauri desktop shell |
| `packages/desktop-electron/package.json` | and `README.md`: Electron desktop shell |

## See Also

- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [UI Client Surfaces](../entities/ui-client-surfaces.md)
- [Desktop and Remote Clients](../entities/desktop-and-remote-clients.md)
- [Workspace Routing and Remote Control](workspace-routing-and-remote-control.md)
- [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
