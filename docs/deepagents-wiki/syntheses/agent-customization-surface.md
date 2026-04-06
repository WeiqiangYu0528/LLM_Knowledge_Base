# Agent Customization Surface

## Overview

This synthesis explains the main extension points that let Deep Agents users shape behavior without rewriting the harness itself.

The value of this synthesis is in showing how Deep Agents composes opinionated defaults with optional extension points instead of treating those as separate products. The point of this page is to make the handoff between systems explicit so a reader can reason about failures, extension points, and policy boundaries without re-deriving the whole path from raw source.

## Systems Involved

| System | Contribution |
| --- | --- |
| [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md) | The planning, filesystem, subagent, and prompt bundle that defines Deep Agents |
| [Filesystem First Agent Configuration](../concepts/filesystem-first-agent-configuration.md) | Skills, memories, and subagents represented as files and folders |
| [Graph Factory](../entities/graph-factory.md) | `create_deep_agent` and the default SDK graph assembly |
| [Example Agents](../entities/example-agents.md) | Examples as applied patterns for research, content, SQL, looping, and remote subagents |
| [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md) | Related page in this wiki. |
| [Architecture Overview](../summaries/architecture-overview.md) | High-level orientation to the monorepo, package boundaries, and execution model |

## Interaction Model

Read this interaction model as a boundary-by-boundary path rather than a single function call:
1. [Graph Factory](../entities/graph-factory.md) contributes one stage of the cross-system path described by this synthesis.
2. [Skills System](../entities/skills-system.md) contributes one stage of the cross-system path described by this synthesis.
3. [Memory System](../entities/memory-system.md) contributes one stage of the cross-system path described by this synthesis.
4. [Subagent System](../entities/subagent-system.md) contributes one stage of the cross-system path described by this synthesis.

What matters here is where ownership changes hands. One subsystem usually receives input, another owns durable state, another chooses policy or execution strategy, and another surfaces the result back to a user or peer system. If those boundaries are not visible, the runtime looks simpler than it actually is and changes become riskier.

## Key Interfaces

| Interface Or Contract | Why It Matters |
| --- | --- |
| `Main `create_deep_agent` assembly logic and default base prompt` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Public SDK export surface` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Skill loading, validation, state schema, and prompt augmentation` | Boundary surface called out by the current page or inferred from the cited sources. |
| `CLI-side skill loading helpers` | Boundary surface called out by the current page or inferred from the cited sources. |
| `Memory source loading, formatting, and prompt injection` | Boundary surface called out by the current page or inferred from the cited sources. |

## Operational Consequences

The synthesis view is usually where debugging stops being guesswork. A symptom that appears in the UI or CLI often originates several layers deeper, especially in repos that separate session state, routing, protocol bridging, plugin activation, and persistence. By following the sequence above, an implementer can decide whether a bug belongs in the initiating surface, the policy layer, the execution layer, or the state-reconciliation layer.

This also explains why change review should span more than one page. A modification that looks local inside an entity page may still alter a synthesis if it changes when state is persisted, which system chooses the active model, how approvals are routed, or where an outbound reply is emitted.

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `libs/deepagents/deepagents/graph.py` | Main `create_deep_agent` assembly logic and default base prompt |
| `libs/deepagents/deepagents/__init__.py` | Public SDK export surface |
| `libs/deepagents/deepagents/middleware/skills.py` | Skill loading, validation, state schema, and prompt augmentation |
| `libs/cli/deepagents_cli/skills/load.py` | CLI-side skill loading helpers |
| `libs/deepagents/deepagents/middleware/memory.py` | Memory source loading, formatting, and prompt injection |
| `deepagents/AGENTS.md` | Monorepo development memory/instructions for contributors |

## See Also

- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Filesystem First Agent Configuration](../concepts/filesystem-first-agent-configuration.md)
- [Graph Factory](../entities/graph-factory.md)
- [Example Agents](../entities/example-agents.md)
- [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
