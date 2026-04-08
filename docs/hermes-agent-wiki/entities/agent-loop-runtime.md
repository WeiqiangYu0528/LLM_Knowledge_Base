# Agent Loop Runtime

## Overview

`run_agent.py` is the runtime center of Hermes Agent. Its `AIAgent` class owns the synchronous conversation loop that every major surface reuses: CLI chat, gateway message handling, ACP editor sessions, cron-triggered prompts, and batch or evaluation runs. The surrounding packages matter, but the decisive architectural fact is that they mostly prepare inputs for `AIAgent` and interpret its callbacks rather than reimplementing agent execution themselves.

That makes this page the hub for the rest of the Hermes wiki. If a behavior affects retries, tool calling, compression, fallback provider selection, memory flushes, or interruption, the implementation usually lives either directly in `run_agent.py` or in a collaborator that `AIAgent` calls at a specific point in the loop.

## Key Types

| Type or method | Role |
|------|------|
| `IterationBudget` in `run_agent.py` | Shared turn-budget object used by parent and child agents |
| `AIAgent` in `run_agent.py` | Core runtime object wrapping model calls, tool execution, callbacks, persistence, and fallback |
| `AIAgent.run_conversation()` | Full entry point returning messages, metadata, and usage data |
| `AIAgent.chat()` | Thin convenience wrapper that extracts the final response string |
| `AIAgent._try_activate_fallback()` | One-shot model/provider failover path |
| `MemoryManager` integration | Hooks built-in and external memory behavior into each turn |

Representative runtime signatures:

```python
class AIAgent:
    def run_conversation(
        self,
        user_message: str,
        system_message=None,
        conversation_history=None,
        task_id=None,
    ) -> dict: ...

    def chat(self, message: str, stream_callback=None) -> str: ...
```

The `chat()` method exists for ergonomic callers. The real runtime behavior is in `run_conversation()`, which appends the new user message, builds the effective prompt, issues provider calls, loops on tool results, persists state, and returns a rich result dict.

## Architecture

`AIAgent` is not a monolithic "do everything inline" loop even though the file is large. It delegates several responsibilities outward:

- prompt construction to `agent/prompt_builder.py`
- compression decisions and summary generation to `agent/context_compressor.py`
- Anthropic prompt caching behavior to `agent/prompt_caching.py`
- provider/model metadata and auxiliary client behavior to `agent/model_metadata.py` and `agent/auxiliary_client.py`
- tool schema discovery and normal registry dispatch to `model_tools.py` and `tools/registry.py`
- persistent memory and provider-specific recall to `agent/memory_manager.py`
- session persistence to `hermes_state.py`

The runtime still keeps several capabilities in-file because they need agent-local state that general tools do not have. The docs call these "agent-level tools": `todo`, `memory`, `session_search`, and `delegate_task`. Their schemas are visible to the model, but the loop intercepts them before the normal registry path because they need access to the current agent state, budget, or session machinery.

The result is a layered architecture:

1. `AIAgent` owns the turn loop and global invariants.
2. Collaborator modules own prompt shaping, provider details, and subsystem-specific helpers.
3. Tool modules own actual capability execution.
4. Surface shells such as CLI or gateway own transport, presentation, and approval UX.

## Runtime Behavior

### Core turn lifecycle

The implementation described in `website/docs/developer-guide/agent-loop.md` matches the code layout in `run_agent.py`:

1. Generate or accept a `task_id`.
2. Append the incoming user message to conversation history.
3. Build or reuse the cached system prompt.
4. Check context pressure and run preflight compression if needed.
5. Format messages for the current `api_mode`.
6. Apply ephemeral overlays such as budget warnings or gateway-specific additions.
7. Make the provider call through an interrupt-aware wrapper.
8. Parse the result.
9. If there are tool calls, execute them and loop.
10. If there is a final response, persist messages, flush memory if needed, and return.

This flow is what keeps CLI, gateway, and ACP behavior aligned. The caller can change callbacks, session history, or working directory, but it still ends up traversing the same turn machine.

### API-mode branching

Hermes supports three API transport shapes:

- `chat_completions`
- `codex_responses`
- `anthropic_messages`

The provider runtime chooses which one is active, but `AIAgent` has to normalize them back to one internal message format. Internally Hermes stores OpenAI-style `role` / `content` / `tool_calls` dicts. That means transport-specific differences are mostly pushed to the edges: formatting input before the call and translating output afterward.

### Tool execution and concurrency

Tool results can be executed either sequentially or concurrently:

- one tool call: execute inline
- multiple tool calls: execute through `ThreadPoolExecutor`
- interactive or approval-sensitive tools can force sequential behavior

Results are reinserted in the original tool-call order even if they complete out of order. This matters because the model relies on consistent tool-result ordering when continuing the loop.

### Callback surfaces

`AIAgent` exposes several callback hooks so surfaces can present runtime progress without changing the loop:

| Callback | Typical use |
|------|------|
| `tool_progress_callback` | CLI spinners, gateway progress notices, ACP tool updates |
| `thinking_callback` | User-facing "thinking" status |
| `reasoning_callback` | Display or stream model reasoning content |
| `clarify_callback` | Surface-specific prompt for human clarification |
| `step_callback` | Progress tracking per full turn |
| `stream_delta_callback` | Streaming token display |
| `status_callback` | ACP or gateway state transitions |

### Budgets, fallback, and compression

The loop keeps three important safety mechanisms in the same runtime owner:

- **Iteration budget:** `IterationBudget` warns at roughly 70 percent and 90 percent usage and eventually forces wrap-up.
- **Fallback provider activation:** `_try_activate_fallback()` can swap provider, model, base URL, API mode, and client in place when the primary runtime fails.
- **Compression:** older turns may be summarized once context pressure crosses configured thresholds, with tool-call groups preserved and memory flushed first.

These are not separate subsystems accidentally touching the loop; they are central correctness mechanisms for long-running agents.

## Source Files

| File | Purpose |
|------|---------|
| `run_agent.py` | `IterationBudget`, `AIAgent`, fallback activation, tool loop, callbacks, compression hooks |
| `website/docs/developer-guide/agent-loop.md` | Maintainer-level description of the turn lifecycle and callback surfaces |
| `agent/context_compressor.py` | In-loop compression algorithm used by the agent runtime |
| `agent/prompt_builder.py` | System prompt assembly used before model calls |
| `model_tools.py` | Tool discovery and normal dispatch bridge |
| `hermes_state.py` | Session persistence used at turn boundaries |

## See Also

- [Prompt Assembly System](prompt-assembly-system.md)
- [Provider Runtime](provider-runtime.md)
- [Tool Registry and Dispatch](tool-registry-and-dispatch.md)
- [Gateway Message to Agent Reply Flow](../syntheses/gateway-message-to-agent-reply-flow.md)
