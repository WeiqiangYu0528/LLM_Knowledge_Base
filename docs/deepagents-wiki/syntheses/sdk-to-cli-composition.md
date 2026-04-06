# SDK to CLI Composition

## Overview

This synthesis explains how `deepagents-cli` is built on top of the SDK rather than beside it. The CLI is a policy-and-product layer that configures the base graph factory with extra tools, UI, persistence, and operational concerns.

The value of this synthesis is in showing how Deep Agents composes opinionated defaults with optional extension points instead of treating those as separate products. The point of this page is to make the handoff between systems explicit so a reader can reason about failures, extension points, and policy boundaries without re-deriving the whole path from raw source.

## Systems Involved

| System | Contribution |
| --- | --- |
| [CLI Runtime](../entities/cli-runtime.md) | CLI entry points, agent construction, server graph bootstrapping, and tool selection |
| [Graph Factory](../entities/graph-factory.md) | `create_deep_agent` and the default SDK graph assembly |
| [Monorepo Package Layering](../concepts/monorepo-package-layering.md) | How SDK, CLI, ACP, evals, partners, and examples stack together |
| [Interactive Session Lifecycle](interactive-session-lifecycle.md) | End-to-end flow from CLI startup to persisted thread resume |
| [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md) | The planning, filesystem, subagent, and prompt bundle that defines Deep Agents |
| [Architecture Overview](../summaries/architecture-overview.md) | High-level orientation to the monorepo, package boundaries, and execution model |

## Interaction Model

Read this interaction model as a boundary-by-boundary path rather than a single function call:
1. [Graph Factory](../entities/graph-factory.md) contributes one stage of the cross-system path described by this synthesis.
2. [CLI Runtime](../entities/cli-runtime.md) contributes one stage of the cross-system path described by this synthesis.
3. [CLI Textual UI](../entities/cli-textual-ui.md) contributes one stage of the cross-system path described by this synthesis.
4. [MCP System](../entities/mcp-system.md) contributes one stage of the cross-system path described by this synthesis.

What matters here is where ownership changes hands. One subsystem usually receives input, another owns durable state, another chooses policy or execution strategy, and another surfaces the result back to a user or peer system. If those boundaries are not visible, the runtime looks simpler than it actually is and changes become riskier.

## Key Interfaces

| Interface Or Contract | Why It Matters |
| --- | --- |
| `Main `create_deep_agent` assembly logic and default base prompt` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Public SDK export surface` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Main CLI entry point and startup logic` | Boundary surface called out by the current page or inferred from the cited sources. |
| `CLI agent assembly and shell approval handling` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Main Textual application` | Boundary surface called out by the current page or inferred from the cited sources. |

## Operational Consequences

The synthesis view is usually where debugging stops being guesswork. A symptom that appears in the UI or CLI often originates several layers deeper, especially in repos that separate session state, routing, protocol bridging, plugin activation, and persistence. By following the sequence above, an implementer can decide whether a bug belongs in the initiating surface, the policy layer, the execution layer, or the state-reconciliation layer.

This also explains why change review should span more than one page. A modification that looks local inside an entity page may still alter a synthesis if it changes when state is persisted, which system chooses the active model, how approvals are routed, or where an outbound reply is emitted.

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `libs/deepagents/deepagents/graph.py` | Main `create_deep_agent` assembly logic and default base prompt |
| `libs/deepagents/deepagents/__init__.py` | Public SDK export surface |
| `libs/cli/deepagents_cli/main.py` | Main CLI entry point and startup logic |
| `libs/cli/deepagents_cli/agent.py` | CLI agent assembly and shell approval handling |
| `libs/cli/deepagents_cli/app.py` | Main Textual application |
| `libs/cli/deepagents_cli/widgets/messages.py` | Chat message rendering widgets |

## See Also

- [CLI Runtime](../entities/cli-runtime.md)
- [Graph Factory](../entities/graph-factory.md)
- [Monorepo Package Layering](../concepts/monorepo-package-layering.md)
- [Interactive Session Lifecycle](interactive-session-lifecycle.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
