# Batteries Included Agent Architecture

## Overview

Deep Agents is built around the idea that useful agent behavior should not require each user to independently discover the same stack. The project therefore bundles planning, filesystem access, shell execution, subagent delegation, and prompt guidance as defaults.

This concept is easier to understand when read as a runtime guarantee rather than a marketing phrase: it describes how Deep Agents keeps customization and execution coherent across SDK, CLI, and remote backends. Batteries Included Agent Architecture is therefore best read as a mechanism with operational consequences, not merely a label for related features.

## Why It Exists

This concept exists because the Deep Agents codebase repeatedly needs a stable way to coordinate behavior across multiple entities without turning those entities into one monolith.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

## Mechanism

A useful way to read this mechanism is as an ordered path through the participating subsystems:
1. a default model and base system prompt.
2. planning support through todo middleware.
3. filesystem and execution tooling.
4. subagent support through the `task` tool.
5. summarization and patching middleware.
6. optional skills and memory layers.
7. The pattern is most explicit in `create_deep_agent`, which assembles: 1.
8. a default model and base system prompt 2.
9. planning support through todo middleware 3.
10. filesystem and execution tooling 4.

The steps above are the operational skeleton. The exact file names vary by subsystem, but the concept remains stable because each participating entity contributes one predictable part of the chain. That is why the same concept can show up in SDK code, CLI wiring, plugin activation, channel routing, or persistence without becoming ambiguous.

## Invariants and Operational Implications

The most important invariant is that the linked entities are allowed to change implementation detail without changing the high-level guarantee described here. When a change breaks that guarantee, the failure usually appears at subsystem boundaries first: a summary no longer compacts correctly, a session route stops being stable, a skill path is not loaded consistently, or a permission rule is evaluated in the wrong layer.

Operationally, this means debugging should follow the mechanism rather than a UI symptom. Start where the concept is introduced, then inspect the next boundary where data, policy, or control is handed off. The source evidence table below is organized to support exactly that style of investigation.

## Involved Entities

| Entity | Role In This Concept |
| --- | --- |
| [Agent Customization Surface](../syntheses/agent-customization-surface.md) | How tools, prompts, skills, memory, subagents, and backends compose |
| [Graph Factory](../entities/graph-factory.md) | `create_deep_agent` and the default SDK graph assembly |
| [Evals System](../entities/evals-system.md) | Behavioral eval framework, category taxonomy, and radar reporting |
| [SDK to CLI Composition](../syntheses/sdk-to-cli-composition.md) | How the CLI extends and configures the base SDK graph |
| [Architecture Overview](../summaries/architecture-overview.md) | High-level orientation to the monorepo, package boundaries, and execution model |
| [Codebase Map](../summaries/codebase-map.md) | Maps `libs/`, `partners/`, and `examples/` to the corresponding wiki pages |

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `libs/deepagents/deepagents/graph.py` | Main `create_deep_agent` assembly logic and default base prompt |
| `libs/deepagents/deepagents/__init__.py` | Public SDK export surface |
| `libs/deepagents/deepagents/backends/protocol.py` | Backend and execution protocol definitions plus result types |
| `libs/deepagents/deepagents/backends/filesystem.py` | Concrete filesystem backend with optional virtual path mode |
| `libs/deepagents/deepagents/middleware/subagents.py` | Core subagent specs, tool description, and middleware integration |
| `libs/deepagents/deepagents/middleware/async_subagents.py` | Remote/background subagent handling |

## See Also

- [Agent Customization Surface](../syntheses/agent-customization-surface.md)
- [Graph Factory](../entities/graph-factory.md)
- [Evals System](../entities/evals-system.md)
- [SDK to CLI Composition](../syntheses/sdk-to-cli-composition.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
