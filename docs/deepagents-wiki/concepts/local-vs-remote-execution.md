# Local vs Remote Execution

## Overview

One of Deep Agents' main architectural strengths is that the agent shape stays fairly stable while the execution substrate changes. The same high-level behavior can target local disk, local shell, remote sandboxes, or fully remote async subagents.

This concept is easier to understand when read as a runtime guarantee rather than a marketing phrase: it describes how Deep Agents keeps customization and execution coherent across SDK, CLI, and remote backends. Local vs Remote Execution is therefore best read as a mechanism with operational consequences, not merely a label for related features.

## Why It Exists

This concept exists because the Deep Agents codebase repeatedly needs a stable way to coordinate behavior across multiple entities without turning those entities into one monolith.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

## Mechanism

A useful way to read this mechanism is as an ordered path through the participating subsystems:
1. The enabler is the [Backend System](../entities/backend-system.md).
2. Filesystem operations and shell execution are abstracted behind protocols, while the CLI and examples decide which backend or provider to instantiate.
3. Async subagents and ACP add another layer by moving whole agent runs behind a network boundary.
4. Remote execution is therefore not a separate product.

The steps above are the operational skeleton. The exact file names vary by subsystem, but the concept remains stable because each participating entity contributes one predictable part of the chain. That is why the same concept can show up in SDK code, CLI wiring, plugin activation, channel routing, or persistence without becoming ambiguous.

## Invariants and Operational Implications

The most important invariant is that the linked entities are allowed to change implementation detail without changing the high-level guarantee described here. When a change breaks that guarantee, the failure usually appears at subsystem boundaries first: a summary no longer compacts correctly, a session route stops being stable, a skill path is not loaded consistently, or a permission rule is evaluated in the wrong layer.

Operationally, this means debugging should follow the mechanism rather than a UI symptom. Start where the concept is introduced, then inspect the next boundary where data, policy, or control is handed off. The source evidence table below is organized to support exactly that style of investigation.

## Involved Entities

| Entity | Role In This Concept |
| --- | --- |
| [Filesystem First Agent Configuration](filesystem-first-agent-configuration.md) | Skills, memories, and subagents represented as files and folders |
| [Backend System](../entities/backend-system.md) | Pluggable storage and execution backends, including composite routing |
| [Sandbox Partners](../entities/sandbox-partners.md) | Daytona, Modal, Runloop, and QuickJS integrations |
| [Remote Subagent and Sandbox Flow](../syntheses/remote-subagent-and-sandbox-flow.md) | The path through remote backends, async tasks, ACP, and hosted servers |
| [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md) | The planning, filesystem, subagent, and prompt bundle that defines Deep Agents |
| [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md) | Related page in this wiki. |

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `libs/deepagents/deepagents/backends/protocol.py` | Backend and execution protocol definitions plus result types |
| `libs/deepagents/deepagents/backends/filesystem.py` | Concrete filesystem backend with optional virtual path mode |
| `libs/deepagents/deepagents/middleware/subagents.py` | Core subagent specs, tool description, and middleware integration |
| `libs/deepagents/deepagents/middleware/async_subagents.py` | Remote/background subagent handling |
| `libs/cli/deepagents_cli/main.py` | Main CLI entry point and startup logic |
| `libs/cli/deepagents_cli/agent.py` | CLI agent assembly and shell approval handling |

## See Also

- [Filesystem First Agent Configuration](filesystem-first-agent-configuration.md)
- [Backend System](../entities/backend-system.md)
- [Sandbox Partners](../entities/sandbox-partners.md)
- [Remote Subagent and Sandbox Flow](../syntheses/remote-subagent-and-sandbox-flow.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
