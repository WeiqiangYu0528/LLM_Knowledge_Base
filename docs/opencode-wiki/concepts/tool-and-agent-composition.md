# Tool and Agent Composition

## Overview

OpenCode's product behavior is shaped by the interaction of built-in agents, prompt variants, and the tool registry. The runtime is not just "a model with tools"; it is a compositional system where modes and capabilities change the effective agent.

This concept exists because OpenCode reuses one runtime across multiple entry points. The concept page therefore needs to explain the shared mechanism that keeps those surfaces aligned. Tool and Agent Composition is therefore best read as a mechanism with operational consequences, not merely a label for related features.

## Why It Exists

This concept exists because the OpenCode codebase repeatedly needs a stable way to coordinate behavior across multiple entities without turning those entities into one monolith.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

A concept page earns its place only when it explains a guarantee that several entities rely on. In OpenCode, this pattern keeps implementation detail from leaking across subsystem boundaries while still letting the overall product behave as one runtime.

## Mechanism

A useful way to read this mechanism is as an ordered path through the participating subsystems:
1. The README highlights built-in build and plan agents plus a general subagent.
2. In code, session prompts vary by mode/model family, the tool registry selectively exposes tools based on model/features, and permissions govern whether some calls proceed automatically or require approval.
3. This means agent behavior is the result of: - prompt variant - tool availability - provider/model choice - permission rules - session state.

The steps above are the operational skeleton. The exact file names vary by subsystem, but the concept remains stable because each participating entity contributes one predictable part of the chain. That is why the same concept can show up in SDK code, CLI wiring, plugin activation, channel routing, or persistence without becoming ambiguous.

## Invariants and Operational Implications

The most important invariant is that the linked entities are allowed to change implementation detail without changing the high-level guarantee described here. When a change breaks that guarantee, the failure usually appears at subsystem boundaries first: a summary no longer compacts correctly, a session route stops being stable, a skill path is not loaded consistently, or a permission rule is evaluated in the wrong layer.

Operationally, this means debugging should follow the mechanism rather than a UI symptom. Start where the concept is introduced, then inspect the next boundary where data, policy, or control is handed off. The source evidence table below is organized to support exactly that style of investigation.

## Involved Entities

| Entity | Role In This Concept |
| --- | --- |
| [Provider Agnostic Model Routing](provider-agnostic-model-routing.md) | Uniform model selection across many provider SDKs |
| [Context Compaction and Session Recovery](../syntheses/context-compaction-and-session-recovery.md) | Prompt variants, summaries, compaction, retry, and restore behavior |
| [Tool System](../entities/tool-system.md) | Tool registry, built-in tool catalog, plugin tools, and model-sensitive tool selection |
| [Session System](../entities/session-system.md) | Session records, prompt selection, messages, compaction, retries, and summaries |
| [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md) | OpenCode core service with multiple clients layered on top |
| [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md) | Related page in this wiki. |

## Source Evidence

| File | Why It Matters |
| --- | --- |
| `packages/opencode/src/session/index.ts` | Session records, events, plan-file path logic, and usage aggregation |
| `packages/opencode/src/session/prompt.ts` | Prompt selection logic |
| `packages/opencode/src/tool/registry.ts` | Dynamic tool assembly and filtering |
| `packages/opencode/src/tool/registry.ts, tool.ts, bash.ts, read.ts, write.ts, task.ts` | Built-in tool implementations |
| `packages/opencode/src/provider/provider.ts` | Provider and model construction logic |
| `packages/opencode/src/provider/schema.ts` | Provider/model identifiers and schemas |

## See Also

- [Provider Agnostic Model Routing](provider-agnostic-model-routing.md)
- [Context Compaction and Session Recovery](../syntheses/context-compaction-and-session-recovery.md)
- [Tool System](../entities/tool-system.md)
- [Session System](../entities/session-system.md)
- [Client Server Agent Architecture](../concepts/client-server-agent-architecture.md)
- [Request To Session Execution Flow](../syntheses/request-to-session-execution-flow.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
