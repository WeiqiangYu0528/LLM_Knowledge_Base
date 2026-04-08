# Interruption and Human Approval Flow

## Overview

Hermes is designed for long-running and tool-heavy sessions, which means it needs explicit control paths for stopping work, queueing new work, and approving dangerous actions. Those control paths differ by surface, but the runtime intent is consistent: users must be able to regain control without corrupting session state or silently allowing risky operations.

## Mechanism

There are two related mechanisms:

### Interruption

1. A session is actively running in the agent loop.
2. A new message or control command arrives.
3. The surface-specific shell decides whether to queue, interrupt, or bypass.
4. The agent loop sees the interrupt event and abandons the in-flight provider call if needed.
5. The session continues with either the queued message or a control action such as stop.

### Approval

1. A tool request reaches an approval-sensitive path, usually the terminal tool.
2. `DANGEROUS_PATTERNS` matches the command or operation.
3. Hermes routes the decision through a surface-specific callback.
4. The user can allow once, deny, or sometimes allow permanently.
5. The result is reinjected so the agent either proceeds or adapts.

The key design is that detection happens near the tool runtime, while transport happens near the surface.

## Involved Entities

- [Agent Loop Runtime](../entities/agent-loop-runtime.md)
- [Tool Registry and Dispatch](../entities/tool-registry-and-dispatch.md)
- [Gateway Runtime](../entities/gateway-runtime.md)
- [ACP Adapter](../entities/acp-adapter.md)
- [CLI Runtime](../entities/cli-runtime.md)

## Source Evidence

- `run_agent.py` — interrupt-aware API call wrapper and agent-level stop behavior
- `tools/approval.py` — dangerous-command patterns and decision logic
- `gateway/run.py` and `gateway/platforms/base.py` — running-agent guard and queueing behavior
- `acp_adapter/permissions.py` — ACP approval bridge
- `website/docs/developer-guide/agent-loop.md` and `gateway-internals.md` — surface-specific interruption behavior

## See Also

- [Tool Call Execution and Approval Pipeline](../syntheses/tool-call-execution-and-approval-pipeline.md)
- [Gateway Message to Agent Reply Flow](../syntheses/gateway-message-to-agent-reply-flow.md)
- [ACP Adapter](../entities/acp-adapter.md)
- [Gateway Runtime](../entities/gateway-runtime.md)
