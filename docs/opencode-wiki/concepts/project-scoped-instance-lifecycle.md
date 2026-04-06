# Project Scoped Instance Lifecycle

## Overview

OpenCode organizes much of its runtime state around the current project directory. Services are initialized lazily for that project and disposed when the instance is reloaded or torn down.

This concept exists because OpenCode reuses one runtime across multiple entry points. The concept page therefore needs to explain the shared mechanism that keeps those surfaces aligned. Project Scoped Instance Lifecycle is therefore best read as a mechanism with operational consequences, not merely a label for related features.

## Why It Exists

This concept exists because the OpenCode codebase repeatedly needs a stable way to coordinate behavior across multiple entities without turning those entities into one monolith.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

## Mechanism

A useful way to read this mechanism is as an ordered path through the participating subsystems:
1. `Instance.provide()` resolves a directory, boots or reuses an instance, restores async context, and then runs the requested function.
2. `InstanceBootstrap()` initializes project-local services such as plugins, LSP, file watchers, VCS, and snapshots.
3. Disposal tears down cached state and emits a global event.
4. This gives OpenCode a durable concept of "the current project" that many independent systems can rely on.

The steps above are the operational skeleton. The exact file names vary by subsystem, but the concept remains stable because each participating entity contributes one predictable part of the chain. That is why the same concept can show up in SDK code, CLI wiring, plugin activation, channel routing, or persistence without becoming ambiguous.

## Invariants and Operational Implications

The most important invariant is that the linked entities are allowed to change implementation detail without changing the high-level guarantee described here. When a change breaks that guarantee, the failure usually appears at subsystem boundaries first: a summary no longer compacts correctly, a session route stops being stable, a skill path is not loaded consistently, or a permission rule is evaluated in the wrong layer.

Operationally, this means debugging should follow the mechanism rather than a UI symptom. Start where the concept is introduced, then inspect the next boundary where data, policy, or control is handed off. The source evidence table below is organized to support exactly that style of investigation.

## Involved Entities

| Entity | Role In This Concept |
| --- | --- |
| [Client Server Agent Architecture](client-server-agent-architecture.md) | OpenCode core service with multiple clients layered on top |
| [Project and Instance System](../entities/project-and-instance-system.md) | Project discovery, worktree context, bootstrap, and instance scoping |
| [Workspace Routing and Remote Control](../syntheses/workspace-routing-and-remote-control.md) | Local vs remote workspaces, adaptors, and forwarded requests |
| [Request to Session Execution Flow](../syntheses/request-to-session-execution-flow.md) | How CLI/server requests become session-level model and tool activity |
| [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md) | OpenCode core service with multiple clients layered on top |
| [Architecture Overview](../summaries/architecture-overview.md) | High-level orientation to the OpenCode monorepo, package boundaries, and runtime model |

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `packages/opencode/src/project/instance.ts` | Project-scoped instance context, caching, reload, and disposal |
| `packages/opencode/src/project/bootstrap.ts` | Bootstrap sequence for plugins, file watchers, VCS, and snapshots |
| `packages/opencode/src/tool/registry.ts` | Dynamic tool assembly and filtering |
| `packages/opencode/src/tool/registry.ts, tool.ts, bash.ts, read.ts, write.ts, task.ts` | Built-in tool implementations |
| `packages/opencode/src/plugin/index.ts` | Plugin runtime, internal plugins, hook triggering, and external loading |
| `packages/opencode/src/plugin/loader.ts` | Plugin loading and resolution |

## See Also

- [Client Server Agent Architecture](client-server-agent-architecture.md)
- [Project and Instance System](../entities/project-and-instance-system.md)
- [Workspace Routing and Remote Control](../syntheses/workspace-routing-and-remote-control.md)
- [Request to Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
