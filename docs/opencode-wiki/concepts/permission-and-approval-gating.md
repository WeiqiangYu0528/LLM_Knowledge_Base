# Permission and Approval Gating

## Overview

OpenCode has an explicit runtime permission model for tool execution. Permissions are neither purely static nor purely UI-based; they are evaluated, requested, persisted, and sometimes promoted into future allow rules.

This concept exists because OpenCode reuses one runtime across multiple entry points. The concept page therefore needs to explain the shared mechanism that keeps those surfaces aligned. Permission and Approval Gating is therefore best read as a mechanism with operational consequences, not merely a label for related features.

## Why It Exists

This concept exists because the OpenCode codebase repeatedly needs a stable way to coordinate behavior across multiple entities without turning those entities into one monolith.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

## Mechanism

A useful way to read this mechanism is as an ordered path through the participating subsystems:
1. Each permission request includes a permission name, patterns, metadata, optional tool-call linkage, and a candidate ruleset.
2. The system evaluates all relevant rules.
3. If nothing denies the request but not everything is allowed, it emits a permission event and waits for a reply.
4. Replies can: - allow once - allow always - reject The "always" path mutates the approved ruleset for future evaluations.

The steps above are the operational skeleton. The exact file names vary by subsystem, but the concept remains stable because each participating entity contributes one predictable part of the chain. That is why the same concept can show up in SDK code, CLI wiring, plugin activation, channel routing, or persistence without becoming ambiguous.

## Invariants and Operational Implications

The most important invariant is that the linked entities are allowed to change implementation detail without changing the high-level guarantee described here. When a change breaks that guarantee, the failure usually appears at subsystem boundaries first: a summary no longer compacts correctly, a session route stops being stable, a skill path is not loaded consistently, or a permission rule is evaluated in the wrong layer.

Operationally, this means debugging should follow the mechanism rather than a UI symptom. Start where the concept is introduced, then inspect the next boundary where data, policy, or control is handed off. The source evidence table below is organized to support exactly that style of investigation.

## Involved Entities

| Entity | Role In This Concept |
| --- | --- |
| [Tool and Agent Composition](tool-and-agent-composition.md) | Build/plan agents, task delegation, and tool orchestration |
| [Permission System](../entities/permission-system.md) | Ask/allow/deny rules, permission requests, and approval lifecycle |
| [Provider Tool Plugin Interaction Model](../syntheses/provider-tool-plugin-interaction-model.md) | How providers, tools, plugins, and permissions shape execution |
| [Request to Session Execution Flow](../syntheses/request-to-session-execution-flow.md) | How CLI/server requests become session-level model and tool activity |
| [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md) | OpenCode core service with multiple clients layered on top |
| [Architecture Overview](../summaries/architecture-overview.md) | High-level orientation to the OpenCode monorepo, package boundaries, and runtime model |

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `packages/opencode/src/permission/index.ts` | Permission rules, request lifecycle, and service implementation |
| `packages/opencode/src/permission/evaluate.ts` | Rule evaluation logic |
| `packages/opencode/src/tool/registry.ts` | Dynamic tool assembly and filtering |
| `packages/opencode/src/tool/registry.ts, tool.ts, bash.ts, read.ts, write.ts, task.ts` | Built-in tool implementations |
| `packages/opencode/src/mcp/index.ts` | MCP client/service implementation, tool conversion, auth, and status tracking |
| `packages/opencode/src/mcp/auth.ts` | MCP auth support |

## See Also

- [Tool and Agent Composition](tool-and-agent-composition.md)
- [Permission System](../entities/permission-system.md)
- [Provider Tool Plugin Interaction Model](../syntheses/provider-tool-plugin-interaction-model.md)
- [Request to Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
