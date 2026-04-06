# Plugin Driven Extensibility

## Overview

OpenCode treats extensibility as a first-class capability. Plugins can contribute hooks, tools, and auth behaviors, and they are loaded into project-scoped runtime state rather than bolted on globally without context.

This concept exists because OpenCode reuses one runtime across multiple entry points. The concept page therefore needs to explain the shared mechanism that keeps those surfaces aligned. Plugin Driven Extensibility is therefore best read as a mechanism with operational consequences, not merely a label for related features.

## Why It Exists

This concept exists because the OpenCode codebase repeatedly needs a stable way to coordinate behavior across multiple entities without turning those entities into one monolith.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

## Mechanism

A useful way to read this mechanism is as an ordered path through the participating subsystems:
1. initializes internal plugins.
2. reads configured plugin origins unless pure mode disables them.
3. resolves/installs external plugins.
4. applies server-side plugin hooks.
5. exposes plugin-contributed tools to the registry.
6. forwards bus/config events into plugin hooks.
7. The plugin runtime: 1.
8. initializes internal plugins 2.
9. reads configured plugin origins unless pure mode disables them 3.
10. resolves/installs external plugins 4.

The steps above are the operational skeleton. The exact file names vary by subsystem, but the concept remains stable because each participating entity contributes one predictable part of the chain. That is why the same concept can show up in SDK code, CLI wiring, plugin activation, channel routing, or persistence without becoming ambiguous.

## Invariants and Operational Implications

The most important invariant is that the linked entities are allowed to change implementation detail without changing the high-level guarantee described here. When a change breaks that guarantee, the failure usually appears at subsystem boundaries first: a summary no longer compacts correctly, a session route stops being stable, a skill path is not loaded consistently, or a permission rule is evaluated in the wrong layer.

Operationally, this means debugging should follow the mechanism rather than a UI symptom. Start where the concept is introduced, then inspect the next boundary where data, policy, or control is handed off. The source evidence table below is organized to support exactly that style of investigation.

## Involved Entities

| Entity | Role In This Concept |
| --- | --- |
| [Client Server Agent Architecture](client-server-agent-architecture.md) | OpenCode core service with multiple clients layered on top |
| [Plugin System](../entities/plugin-system.md) | Internal and external plugin loading, hooks, and SDK-backed plugin context |
| [Provider Tool Plugin Interaction Model](../syntheses/provider-tool-plugin-interaction-model.md) | How providers, tools, plugins, and permissions shape execution |
| [Multi Client Product Architecture](../syntheses/multi-client-product-architecture.md) | TUI, app, web, desktop, Slack, and SDK surfaces around one core |
| [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md) | OpenCode core service with multiple clients layered on top |
| [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md) | Related page in this wiki. |

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `packages/opencode/src/plugin/index.ts` | Plugin runtime, internal plugins, hook triggering, and external loading |
| `packages/opencode/src/plugin/loader.ts` | Plugin loading and resolution |
| `packages/opencode/src/tool/registry.ts` | Dynamic tool assembly and filtering |
| `packages/opencode/src/tool/registry.ts, tool.ts, bash.ts, read.ts, write.ts, task.ts` | Built-in tool implementations |
| `packages/opencode/src/provider/provider.ts` | Provider and model construction logic |
| `packages/opencode/src/provider/schema.ts` | Provider/model identifiers and schemas |

## See Also

- [Client Server Agent Architecture](client-server-agent-architecture.md)
- [Plugin System](../entities/plugin-system.md)
- [Provider Tool Plugin Interaction Model](../syntheses/provider-tool-plugin-interaction-model.md)
- [Multi Client Product Architecture](../syntheses/multi-client-product-architecture.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
