# CLI Runtime

## Overview

The CLI is Hermes' most visible surface, but it is not just a thin terminal wrapper around `AIAgent`. `hermes_cli/main.py` owns startup ordering, profile overrides, environment loading, command parsing, setup and configuration entry points, and the default interactive runtime. It is also the place where several non-chat surfaces are launched, including gateway management, cron management, model switching, ACP startup, and session browsing.

## Key Types / Key Concepts

| Type or module | Role |
|------|------|
| `main()` in `hermes_cli/main.py` | Entrypoint for the `hermes` console script |
| `_apply_profile_override()` | Pre-import profile selection that rewrites `HERMES_HOME` before modules cache it |
| `COMMAND_REGISTRY` in `hermes_cli/commands.py` | Canonical slash-command registry shared by CLI and gateway |
| `hermes_cli/setup.py` | Interactive setup wizard and guided configuration |
| `hermes_cli/models.py`, `model_switch.py` | Model discovery and runtime switching surfaces |

## Architecture

The CLI runtime has to do two things at once:

- behave like a user-facing application shell
- initialize shared runtime state correctly for every downstream subsystem

That is why `hermes_cli/main.py` does critical work before it imports much of the rest of the repo. `_apply_profile_override()` must run before other Hermes modules import so that the correct `HERMES_HOME` value is visible everywhere. After that, the CLI loads `.env`, sets up logging, resolves configuration, and then dispatches into subcommands or the interactive chat shell.

## Runtime Behavior

### Startup order

The most important CLI invariant is "profile selection first". Many modules read `HERMES_HOME` at import time, so the CLI strips `--profile` or `-p` out of `sys.argv`, resolves the target profile, sets the environment variable, and only then continues normal imports.

After that it:

1. loads `~/.hermes/.env`
2. initializes file logging
3. imports shared runtime modules
4. parses commands and subcommands
5. either enters interactive chat or dispatches a management surface such as setup, gateway, cron, ACP, or sessions

### Slash-command infrastructure

The CLI and gateway share `COMMAND_REGISTRY` from `hermes_cli/commands.py`. This is a subtle but important design decision: help text, autocomplete, gateway-known commands, and plugin-added commands all derive from one registry rather than duplicated lists. The CLI owns the interactive presentation, but command identity is a shared runtime concern.

### Configuration and setup surfaces

Many features that look like "separate tools" are really part of the CLI runtime:

- `hermes setup`
- `hermes config ...`
- `hermes model ...`
- `hermes tools ...`
- `hermes skills ...`
- `hermes cron ...`
- `hermes gateway ...`
- `hermes acp`

That makes `hermes_cli/` a control plane package for the whole product rather than a narrow REPL wrapper.

## Source Files

| File | Purpose |
|------|---------|
| `hermes_cli/main.py` | Console entrypoint, profile override, env loading, command dispatch |
| `hermes_cli/commands.py` | Shared slash-command registry and derived lookups |
| `hermes_cli/setup.py` | Setup wizard and first-run configuration |
| `hermes_cli/models.py` | Model catalog views and selection helpers |
| `hermes_cli/model_switch.py` | Shared model switching flow across CLI and gateway |

## See Also

- [Config and Profile System](config-and-profile-system.md)
- [Provider Runtime](provider-runtime.md)
- [CLI to Agent Loop Composition](../syntheses/cli-to-agent-loop-composition.md)
- [Gateway Runtime](gateway-runtime.md)
