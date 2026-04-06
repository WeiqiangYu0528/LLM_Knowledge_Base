# Remote Subagent and Sandbox Flow

## Overview

This synthesis describes the repo's remote-execution story: how work can move from the main agent into remote sandboxes, async subagents, or editor-hosted servers while keeping a familiar Deep Agents interface.

The value of this synthesis is in showing how Deep Agents composes opinionated defaults with optional extension points instead of treating those as separate products. The point of this page is to make the handoff between systems explicit so a reader can reason about failures, extension points, and policy boundaries without re-deriving the whole path from raw source.

## Systems Involved

| System | Contribution |
| --- | --- |
| [Local vs Remote Execution](../concepts/local-vs-remote-execution.md) | Local shell execution, remote sandboxes, and async remote tasks |
| [ACP Server](../entities/acp-server.md) | Agent Client Protocol bridge for editor-hosted Deep Agents |
| [Sandbox Partners](../entities/sandbox-partners.md) | Daytona, Modal, Runloop, and QuickJS integrations |
| [Subagent System](../entities/subagent-system.md) | Declarative, compiled, and async subagents exposed via the `task` tool |
| [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md) | The planning, filesystem, subagent, and prompt bundle that defines Deep Agents |
| [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md) | Related page in this wiki. |

## Interaction Model

Read this interaction model as a boundary-by-boundary path rather than a single function call:
1. [Backend System](../entities/backend-system.md) contributes one stage of the cross-system path described by this synthesis.
2. [Subagent System](../entities/subagent-system.md) contributes one stage of the cross-system path described by this synthesis.
3. [Sandbox Partners](../entities/sandbox-partners.md) contributes one stage of the cross-system path described by this synthesis.
4. [CLI Runtime](../entities/cli-runtime.md) contributes one stage of the cross-system path described by this synthesis.

What matters here is where ownership changes hands. One subsystem usually receives input, another owns durable state, another chooses policy or execution strategy, and another surfaces the result back to a user or peer system. If those boundaries are not visible, the runtime looks simpler than it actually is and changes become riskier.

## Key Interfaces

| Interface Or Contract | Why It Matters |
| --- | --- |
| `Backend and execution protocol definitions plus result types` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Concrete filesystem backend with optional virtual path mode` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Core subagent specs, tool description, and middleware integration` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Remote/background subagent handling` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Daytona sandbox package` | Boundary surface called out by the current page or inferred from the cited sources. |

## Operational Consequences

The synthesis view is usually where debugging stops being guesswork. A symptom that appears in the UI or CLI often originates several layers deeper, especially in repos that separate session state, routing, protocol bridging, plugin activation, and persistence. By following the sequence above, an implementer can decide whether a bug belongs in the initiating surface, the policy layer, the execution layer, or the state-reconciliation layer.

This also explains why change review should span more than one page. A modification that looks local inside an entity page may still alter a synthesis if it changes when state is persisted, which system chooses the active model, how approvals are routed, or where an outbound reply is emitted.

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `libs/deepagents/deepagents/backends/protocol.py` | Backend and execution protocol definitions plus result types |
| `libs/deepagents/deepagents/backends/filesystem.py` | Concrete filesystem backend with optional virtual path mode |
| `libs/deepagents/deepagents/middleware/subagents.py` | Core subagent specs, tool description, and middleware integration |
| `libs/deepagents/deepagents/middleware/async_subagents.py` | Remote/background subagent handling |
| `libs/partners/daytona/README.md` | Daytona sandbox package |
| `libs/partners/modal/README.md` | Modal sandbox package |

## See Also

- [Local vs Remote Execution](../concepts/local-vs-remote-execution.md)
- [ACP Server](../entities/acp-server.md)
- [Sandbox Partners](../entities/sandbox-partners.md)
- [Subagent System](../entities/subagent-system.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
