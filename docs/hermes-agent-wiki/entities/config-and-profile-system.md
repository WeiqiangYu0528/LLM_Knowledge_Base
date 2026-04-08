# Config and Profile System

## Overview

Hermes is heavily shaped by profile-scoped filesystem state. `HERMES_HOME` determines where configuration, secrets, identity, skills, memories, cron state, logs, and sandbox artifacts live. The config and profile subsystem therefore exists to make runtime state reproducible across CLI, gateway, ACP, and background surfaces without hardcoding everything into process-local memory.

## Key Types / Key Concepts

| Function or concept | Role |
|------|------|
| `get_hermes_home()` | Canonical profile-scoped home directory resolver |
| `_apply_profile_override()` in `hermes_cli/main.py` | Pre-import profile switch logic |
| `DEFAULT_CONFIG` in `hermes_cli/config.py` | Baseline config structure |
| `config.yaml` | Main structured runtime config |
| `.env` | Secret storage for provider keys and related tokens |

## Architecture

The config system is distributed across:

- `hermes_cli/main.py` for very early profile override
- `hermes_cli/config.py` for path helpers, defaults, config loading, and managed-install behavior
- `hermes_constants.py` for canonical path helpers reused elsewhere
- the setup wizard and model/provider commands for higher-level user flows

The critical boundary is that profile selection has to happen before most imports, while normal config reading can happen later.

## Runtime Behavior

### Profile override before imports

`_apply_profile_override()` parses `--profile` and even consults `~/.hermes/active_profile` before the rest of Hermes loads. This is not cosmetic. Many modules read home-directory paths at import time, so the CLI must set `HERMES_HOME` first to prevent stale paths from being cached.

### Files and directories owned by the home directory

`ensure_hermes_home()` creates a profile-local directory tree including:

- `config.yaml`
- `.env`
- `SOUL.md`
- `cron/`
- `sessions/`
- `logs/`
- `memories/`

That makes the home directory the effective runtime root for user state.

### Managed-install behavior

Hermes can run in managed modes such as Homebrew or NixOS. `hermes_cli/config.py` includes helpers that detect those environments and emit more constrained guidance, such as preferring package-manager update flows over local mutation. This matters because config mutation and installation behavior are not always symmetrical between source checkouts and package-managed installs.

## Source Files

| File | Purpose |
|------|---------|
| `hermes_cli/main.py` | Profile override and startup ordering |
| `hermes_cli/config.py` | Config defaults, file paths, managed-mode checks, home-directory initialization |
| `hermes_constants.py` | Canonical profile-aware path helpers |
| `hermes_cli/setup.py` | Guided configuration surface |

## See Also

- [CLI Runtime](cli-runtime.md)
- [Provider Runtime](provider-runtime.md)
- [Prompt Assembly System](prompt-assembly-system.md)
- [Multi-Surface Session Continuity](../concepts/multi-surface-session-continuity.md)
