# Monorepo Package Layering

## Overview

The Deep Agents repository is a layered monorepo rather than a single package. The packages are independently meaningful, but they are arranged so that the SDK is foundational and the other packages either expose it, extend it, or validate it.

This concept is easier to understand when read as a runtime guarantee rather than a marketing phrase: it describes how Deep Agents keeps customization and execution coherent across SDK, CLI, and remote backends. Monorepo Package Layering is therefore best read as a mechanism with operational consequences, not merely a label for related features.

## Why It Exists

This concept exists because the Deep Agents codebase repeatedly needs a stable way to coordinate behavior across multiple entities without turning those entities into one monolith.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

## Mechanism

A useful way to read this mechanism is as an ordered path through the participating subsystems:
1. Monorepo Package Layering begins in one or more of the linked entity pages.
2. Runtime code applies the governing rules or conventions described by the concept.
3. Neighboring systems rely on the resulting guarantees when they compose with one another.
4. The concept becomes visible through the source evidence and cross-system syntheses linked below.

The steps above are the operational skeleton. The exact file names vary by subsystem, but the concept remains stable because each participating entity contributes one predictable part of the chain. That is why the same concept can show up in SDK code, CLI wiring, plugin activation, channel routing, or persistence without becoming ambiguous.

## Invariants and Operational Implications

The most important invariant is that the linked entities are allowed to change implementation detail without changing the high-level guarantee described here. When a change breaks that guarantee, the failure usually appears at subsystem boundaries first: a summary no longer compacts correctly, a session route stops being stable, a skill path is not loaded consistently, or a permission rule is evaluated in the wrong layer.

Operationally, this means debugging should follow the mechanism rather than a UI symptom. Start where the concept is introduced, then inspect the next boundary where data, policy, or control is handed off. The source evidence table below is organized to support exactly that style of investigation.

## Involved Entities

| Entity | Role In This Concept |
| --- | --- |
| [Architecture Overview](../summaries/architecture-overview.md) | High-level orientation to the monorepo, package boundaries, and execution model |
| [SDK to CLI Composition](../syntheses/sdk-to-cli-composition.md) | How the CLI extends and configures the base SDK graph |
| [Graph Factory](../entities/graph-factory.md) | `create_deep_agent` and the default SDK graph assembly |
| [Example Agents](../entities/example-agents.md) | Examples as applied patterns for research, content, SQL, looping, and remote subagents |
| [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md) | The planning, filesystem, subagent, and prompt bundle that defines Deep Agents |
| [Codebase Map](../summaries/codebase-map.md) | Maps `libs/`, `partners/`, and `examples/` to the corresponding wiki pages |

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `libs/deepagents/deepagents/graph.py` | Main `create_deep_agent` assembly logic and default base prompt |
| `libs/deepagents/deepagents/__init__.py` | Public SDK export surface |
| `libs/cli/deepagents_cli/main.py` | Main CLI entry point and startup logic |
| `libs/cli/deepagents_cli/agent.py` | CLI agent assembly and shell approval handling |
| `libs/acp/deepagents_acp/server.py` | ACP bridge and session lifecycle implementation |
| `libs/acp/deepagents_acp/utils.py` | Content conversion and protocol helper utilities |

## See Also

- [Architecture Overview](../summaries/architecture-overview.md)
- [SDK to CLI Composition](../syntheses/sdk-to-cli-composition.md)
- [Graph Factory](../entities/graph-factory.md)
- [Example Agents](../entities/example-agents.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Codebase Map](../summaries/codebase-map.md)
