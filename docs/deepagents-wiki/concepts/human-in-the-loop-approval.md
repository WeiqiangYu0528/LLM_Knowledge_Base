# Human in the Loop Approval

## Overview

Deep Agents assumes that some operations should pause for user review, especially when the agent has powerful filesystem or shell access. Human approval is therefore woven into both SDK and CLI design rather than bolted on afterward.

This concept is easier to understand when read as a runtime guarantee rather than a marketing phrase: it describes how Deep Agents keeps customization and execution coherent across SDK, CLI, and remote backends. Human in the Loop Approval is therefore best read as a mechanism with operational consequences, not merely a label for related features.

## Why It Exists

This concept exists because the Deep Agents codebase repeatedly needs a stable way to coordinate behavior across multiple entities without turning those entities into one monolith.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In Deep Agents, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

## Mechanism

A useful way to read this mechanism is as an ordered path through the participating subsystems:
1. At the SDK level, `interrupt_on` config can pause execution before specific tools.
2. In the CLI, `ShellAllowListMiddleware` adds an additional policy layer: disallowed shell commands are rejected before execution in non-interactive contexts, avoiding fragmented traces while still enforcing constraints.
3. The ACP and Textual surfaces then expose these pauses or decisions to users through their respective interfaces.

The steps above are the operational skeleton. The exact file names vary by subsystem, but the concept remains stable because each participating entity contributes one predictable part of the chain. That is why the same concept can show up in SDK code, CLI wiring, plugin activation, channel routing, or persistence without becoming ambiguous.

## Invariants and Operational Implications

The most important invariant is that the linked entities are allowed to change implementation detail without changing the high-level guarantee described here. When a change breaks that guarantee, the failure usually appears at subsystem boundaries first: a summary no longer compacts correctly, a session route stops being stable, a skill path is not loaded consistently, or a permission rule is evaluated in the wrong layer.

Operationally, this means debugging should follow the mechanism rather than a UI symptom. Start where the concept is introduced, then inspect the next boundary where data, policy, or control is handed off. The source evidence table below is organized to support exactly that style of investigation.

## Involved Entities

| Entity | Role In This Concept |
| --- | --- |
| [Local vs Remote Execution](local-vs-remote-execution.md) | Local shell execution, remote sandboxes, and async remote tasks |
| [CLI Runtime](../entities/cli-runtime.md) | CLI entry points, agent construction, server graph bootstrapping, and tool selection |
| [Remote Subagent and Sandbox Flow](../syntheses/remote-subagent-and-sandbox-flow.md) | The path through remote backends, async tasks, ACP, and hosted servers |
| [Interactive Session Lifecycle](../syntheses/interactive-session-lifecycle.md) | End-to-end flow from CLI startup to persisted thread resume |
| [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md) | The planning, filesystem, subagent, and prompt bundle that defines Deep Agents |
| [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md) | Related page in this wiki. |

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

- [Local vs Remote Execution](local-vs-remote-execution.md)
- [CLI Runtime](../entities/cli-runtime.md)
- [Remote Subagent and Sandbox Flow](../syntheses/remote-subagent-and-sandbox-flow.md)
- [Interactive Session Lifecycle](../syntheses/interactive-session-lifecycle.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
