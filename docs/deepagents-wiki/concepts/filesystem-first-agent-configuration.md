# Filesystem First Agent Configuration

## Overview

Deep Agents repeatedly expresses agent behavior through folders and markdown files rather than only through code. Skills live in `SKILL.md`, persistent memory lives in `AGENTS.md`, and CLI-defined subagents are loaded from folder structures.

This concept is easier to understand when read as a runtime guarantee rather than a marketing phrase: it describes how Deep Agents keeps customization and execution coherent across SDK, CLI, and remote backends. Filesystem First Agent Configuration is therefore best read as a mechanism with operational consequences, not merely a label for related features.

## Why It Exists

This concept exists because the Deep Agents codebase repeatedly needs a stable way to coordinate behavior across multiple entities without turning those entities into one monolith.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

## Mechanism

A useful way to read this mechanism is as an ordered path through the participating subsystems:
1. This pattern works because the relevant middleware and loaders read their inputs through the backend abstraction.
2. A skill source is just a path.
3. A memory source is just a path.
4. A project-local subagent is a folder with frontmatter.

The steps above are the operational skeleton. The exact file names vary by subsystem, but the concept remains stable because each participating entity contributes one predictable part of the chain. That is why the same concept can show up in SDK code, CLI wiring, plugin activation, channel routing, or persistence without becoming ambiguous.

## Invariants and Operational Implications

The most important invariant is that the linked entities are allowed to change implementation detail without changing the high-level guarantee described here. When a change breaks that guarantee, the failure usually appears at subsystem boundaries first: a summary no longer compacts correctly, a session route stops being stable, a skill path is not loaded consistently, or a permission rule is evaluated in the wrong layer.

Operationally, this means debugging should follow the mechanism rather than a UI symptom. Start where the concept is introduced, then inspect the next boundary where data, policy, or control is handed off. The source evidence table below is organized to support exactly that style of investigation.

## Involved Entities

| Entity | Role In This Concept |
| --- | --- |
| [Local vs Remote Execution](local-vs-remote-execution.md) | Local shell execution, remote sandboxes, and async remote tasks |
| [Skills System](../entities/skills-system.md) | Skill discovery, metadata parsing, source precedence, and prompt injection |
| [Memory System](../entities/memory-system.md) | `AGENTS.md` loading and persistent memory prompt behavior |
| [Agent Customization Surface](../syntheses/agent-customization-surface.md) | How tools, prompts, skills, memory, subagents, and backends compose |
| [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md) | The planning, filesystem, subagent, and prompt bundle that defines Deep Agents |
| [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md) | Related page in this wiki. |

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `libs/deepagents/deepagents/middleware/skills.py` | Skill loading, validation, state schema, and prompt augmentation |
| `libs/cli/deepagents_cli/skills/load.py` | CLI-side skill loading helpers |
| `libs/deepagents/deepagents/middleware/memory.py` | Memory source loading, formatting, and prompt injection |
| `deepagents/AGENTS.md` | Monorepo development memory/instructions for contributors |
| `libs/deepagents/deepagents/middleware/subagents.py` | Core subagent specs, tool description, and middleware integration |
| `libs/deepagents/deepagents/middleware/async_subagents.py` | Remote/background subagent handling |

## See Also

- [Local vs Remote Execution](local-vs-remote-execution.md)
- [Skills System](../entities/skills-system.md)
- [Memory System](../entities/memory-system.md)
- [Agent Customization Surface](../syntheses/agent-customization-surface.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
