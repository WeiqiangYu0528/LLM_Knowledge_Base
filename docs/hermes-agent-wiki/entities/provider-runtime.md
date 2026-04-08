# Provider Runtime

## Overview

Hermes uses a shared provider runtime so that CLI, gateway, ACP, cron, and auxiliary model tasks all resolve credentials and API transport shapes through one path. This is the reason a user can configure a provider once, then reuse that decision across interactive chat, background execution, editor integration, and memory or compression side tasks.

The core idea is simple: turn a user-facing `(provider, model)` choice plus config and environment data into a concrete client bundle containing provider identity, API mode, base URL, and credentials. The implementation details are complex because Hermes supports many provider families, native Anthropic transport, Codex Responses, custom OpenAI-compatible endpoints, and independent routing for auxiliary work.

## Key Types / Key Concepts

| Function or concept | Role |
|------|------|
| `resolve_provider()` in `hermes_cli/auth.py` | Maps user-facing provider names, aliases, and auth rules to provider metadata |
| `resolve_runtime_provider()` in `hermes_cli/runtime_provider.py` | Builds the concrete runtime bundle used by primary sessions |
| `resolve_provider_client()` in `agent/auxiliary_client.py` | Equivalent resolver for auxiliary tasks and helper model calls |
| `api_mode` | Provider transport mode: `chat_completions`, `codex_responses`, or `anthropic_messages` |
| fallback model | Optional second provider/model pair activated after primary runtime failures |

## Architecture

Provider resolution spans three collaborating modules:

- `hermes_cli/auth.py` owns the registry of known provider families and auth resolution rules.
- `hermes_cli/runtime_provider.py` turns provider selection plus config and environment state into a runtime client bundle.
- `agent/auxiliary_client.py` mirrors the same logic for side channels such as compression summaries, vision, web extraction summarization, session-search summarization, and memory flush work.

This split is important. The CLI shell does not own provider logic even though many users first encounter it there. The gateway and ACP adapter reuse the same runtime resolver so that a model switch or saved custom endpoint behaves consistently across surfaces.

## Runtime Behavior

### Resolution precedence

The provider runtime follows a stable precedence order:

1. explicit runtime request
2. saved `config.yaml` settings
3. environment variables
4. provider defaults and auto-detection

That ordering prevents a stale shell export from silently overriding a provider or model the user already selected via Hermes commands or setup flows.

### API-mode determination

Hermes branches into three transport shapes:

| API mode | Typical use |
|------|------|
| `chat_completions` | OpenAI-compatible endpoints such as OpenRouter, custom endpoints, and many third-party providers |
| `codex_responses` | OpenAI Codex / Responses API |
| `anthropic_messages` | Native Anthropic Messages API |

The runtime chooses an `api_mode` from explicit settings, provider family, or base-URL heuristics. The agent loop then formats messages appropriately and normalizes them back into the internal message shape.

### Custom endpoints and key scoping

Hermes distinguishes between:

- real custom OpenAI-compatible endpoints chosen by the user
- built-in provider paths such as OpenRouter or AI Gateway

It also scopes provider keys so the wrong credential does not leak to the wrong base URL. `OPENROUTER_API_KEY` should not be sent to an arbitrary custom endpoint, and `OPENAI_API_KEY` should not accidentally override a saved OpenRouter selection. This is one of the most important correctness details in the provider runtime because Hermes is intentionally flexible about provider composition.

### Native Anthropic and Codex paths

Anthropic is not treated as merely "OpenAI-compatible with a different URL". When the runtime selects `anthropic`, Hermes moves onto the native `anthropic_messages` path and uses `agent/anthropic_adapter.py` for format translation. Similarly, Codex uses the Responses path rather than pretending to be a normal chat-completions provider.

These branches matter because they affect:

- message formatting
- tool-call shape
- prompt-caching behavior
- error handling
- credential refresh behavior

### Auxiliary routing and fallback

Auxiliary tasks can use either their own provider/model pair or inherit the main runtime. That means operations such as compression summaries or web extraction can be cheaper or differently tuned than the main conversation model while still sharing the same resolver logic.

Fallback is activated from the agent loop, but the provider runtime makes it possible. `_try_activate_fallback()` rebuilds the active client with a new provider/model pair and a new `api_mode`, then swaps it into the live `AIAgent`. Hermes only does this once per conversation so fallback does not ping-pong across providers.

## Source Files

| File | Purpose |
|------|---------|
| `hermes_cli/runtime_provider.py` | Primary runtime resolver and custom-endpoint handling |
| `hermes_cli/auth.py` | Provider registry, aliases, and credential selection |
| `agent/auxiliary_client.py` | Auxiliary model routing and side-channel provider resolution |
| `run_agent.py` | Fallback activation path and integration with the main loop |
| `website/docs/developer-guide/provider-runtime.md` | Maintainer explanation of resolution precedence and transport branching |

## See Also

- [Agent Loop Runtime](agent-loop-runtime.md)
- [CLI Runtime](cli-runtime.md)
- [Tool Registry and Dispatch](tool-registry-and-dispatch.md)
- [CLI to Agent Loop Composition](../syntheses/cli-to-agent-loop-composition.md)
