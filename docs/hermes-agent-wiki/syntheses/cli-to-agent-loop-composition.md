# CLI to Agent Loop Composition

## Overview

This page explains how the `hermes` command-line surface turns filesystem state, config, command routing, toolset selection, and provider resolution into a live `AIAgent` turn. The key point is that the CLI does not merely forward user input to the agent loop. It prepares the runtime environment the loop depends on and owns several preconditions that must be correct before the first model call can happen.

## Systems Involved

- [CLI Runtime](../entities/cli-runtime.md)
- [Config and Profile System](../entities/config-and-profile-system.md)
- [Provider Runtime](../entities/provider-runtime.md)
- [Tool Registry and Dispatch](../entities/tool-registry-and-dispatch.md)
- [Prompt Assembly System](../entities/prompt-assembly-system.md)
- [Agent Loop Runtime](../entities/agent-loop-runtime.md)

## Interaction Model

The composition path is:

1. `hermes_cli/main.py` starts and applies profile override before most Hermes imports.
2. The CLI loads the profile-scoped `.env`, logging setup, and `config.yaml`.
3. Command parsing decides whether the user is entering chat, switching models, managing gateway or cron, or invoking another management path.
4. For interactive chat, the CLI resolves the active provider, model, and toolset configuration.
5. Tool definitions are assembled through `model_tools.py` and the registry.
6. The CLI constructs or resumes session state from SQLite.
7. `AIAgent.run_conversation()` receives the user message along with callbacks, tool definitions, prompt context, and the current session state.
8. Final responses and tool progress are rendered back through the CLI display layer.

This is why CLI bugs often manifest as runtime bugs even when the agent loop is technically correct. Incorrect profile selection or config loading upstream changes the meaning of the entire session.

## Key Interfaces

| Interface | Why it matters |
|------|------|
| `_apply_profile_override()` | Ensures the correct `HERMES_HOME` is visible before module import side effects |
| `load_hermes_dotenv(...)` | Loads secrets and runtime environment from the active profile |
| `COMMAND_REGISTRY` | Shared command identity that affects both CLI and gateway |
| `resolve_runtime_provider(...)` | Produces the concrete model/provider client used by the agent loop |
| `get_tool_definitions(...)` | Determines the actual tool surface shown to the model |

The handoff boundary is especially clear between steps 4 and 7: before that point the CLI is acting as a control plane; after that point `AIAgent` owns the live turn.

## Source Evidence

- `hermes_cli/main.py` — profile override, environment loading, main CLI dispatch
- `hermes_cli/config.py` — path resolution and default config
- `hermes_cli/runtime_provider.py` — provider runtime construction
- `hermes_cli/commands.py` — shared command registry
- `run_agent.py` — final execution loop entered after CLI setup

## See Also

- [CLI Runtime](../entities/cli-runtime.md)
- [Config and Profile System](../entities/config-and-profile-system.md)
- [Agent Loop Runtime](../entities/agent-loop-runtime.md)
- [Prompt Layering and Cache Stability](../concepts/prompt-layering-and-cache-stability.md)
