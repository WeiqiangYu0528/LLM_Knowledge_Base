# Prompt Assembly System

## Overview

Hermes treats prompt assembly as a first-class subsystem because it is where identity, memory, project context, skills, and surface-specific behavior all converge. The key design choice is the separation between a stable cached prompt prefix and ephemeral, API-call-time additions. That separation is what allows Hermes to benefit from Anthropic prompt caching, keep stable identity and memory layers across turns, and avoid mutating the prompt prefix every time a transient runtime condition changes.

## Key Types / Key Concepts

| Function or concept | Role |
|------|------|
| `load_soul_md()` in `agent/prompt_builder.py` | Loads `SOUL.md` from `HERMES_HOME` as the agent identity layer |
| `build_context_files_prompt()` in `agent/prompt_builder.py` | Resolves project context files using a priority order |
| `DEFAULT_AGENT_IDENTITY` | Fallback identity when `SOUL.md` is absent |
| frozen memory snapshot | Built-in memory and user-profile layers injected into the cached prompt |
| ephemeral system prompt | Per-call overlay that is intentionally excluded from the cached prefix |

Representative signatures:

```python
def load_soul_md() -> Optional[str]: ...
def build_context_files_prompt(cwd: Optional[str] = None, skip_soul: bool = False) -> str: ...
```

## Architecture

The prompt system is centered in `agent/prompt_builder.py`, but it draws data from several other owners:

- `hermes_cli/config.py` and runtime config for optional system-message layers
- memory files and the `MemoryManager` for built-in memory snapshots
- `tools/skills_tool.py` and skill-loading surfaces for skill indexes
- project files in the current working directory such as `.hermes.md`, `HERMES.md`, `AGENTS.md`, `CLAUDE.md`, or Cursor rule files
- `agent/prompt_caching.py` for cache breakpoint markers on Anthropic paths

The important ownership line is that prompt assembly decides *what* becomes stable prompt state, while the agent loop decides *when* to rebuild it.

## Runtime Behavior

### Stable prompt layers

The developer guide lays out the cached prompt layers in roughly this order:

1. agent identity from `SOUL.md` or `DEFAULT_AGENT_IDENTITY`
2. tool-aware behavior guidance
3. optional Honcho static block
4. optional system message
5. frozen persistent memory snapshot
6. frozen user profile snapshot
7. skills index
8. project context files
9. timestamp and optional session ID
10. platform hint

This order matters. Identity comes first because it defines the agent persona. Memory and user profile follow because they should behave like stable background facts. Project context is later because it depends on the active working directory. Platform hints come last because they are surface-specific framing rather than core identity.

### Context-file priority

`build_context_files_prompt()` does not merge every supported project-context format. It uses a first-match-wins priority:

1. `.hermes.md` or `HERMES.md`
2. `AGENTS.md`
3. `CLAUDE.md`
4. `.cursorrules` or `.cursor/rules/*.mdc`

That design prevents conflicting instruction files from all being loaded at once. It also means Hermes can remain compatible with several agent ecosystems while still preferring its own project-native context format when present.

### SOUL.md and duplicate suppression

`SOUL.md` is treated specially. When it is loaded as the identity layer, `build_context_files_prompt()` is called with `skip_soul=True` so the same file is not injected again as generic project context. This prevents duplicated persona text and stabilizes the cached prefix.

### Security and truncation

Prompt input from files is not trusted blindly. The prompt builder:

- scans for prompt-injection-like patterns
- strips YAML frontmatter from `.hermes.md`
- truncates oversized context files
- keeps `SOUL.md` and project context under explicit size limits

The point is not strong sandbox security; it is keeping file-based prompt inputs legible, bounded, and less vulnerable to obvious instruction poisoning.

### Ephemeral overlays

Certain layers are intentionally excluded from the cached prefix:

- `ephemeral_system_prompt`
- per-turn budget warnings
- gateway session overlays
- later-turn dynamic recall content
- prefill messages

Those pieces vary too often. If they were merged into the stable prefix, prompt caching would be less effective and previously cached prefixes would churn every turn.

## Source Files

| File | Purpose |
|------|---------|
| `agent/prompt_builder.py` | Core prompt assembly, `SOUL.md` loading, context-file priority, sanitization |
| `agent/prompt_caching.py` | Anthropic cache-marker handling for stable prefixes |
| `website/docs/developer-guide/prompt-assembly.md` | Maintainer documentation for the cached vs ephemeral split |
| `tools/memory_tool.py` | Built-in memory write/read behavior that influences memory snapshots |
| `run_agent.py` | Rebuild timing and injection of ephemeral overlays |

## See Also

- [Agent Loop Runtime](agent-loop-runtime.md)
- [Memory and Learning Loop](memory-and-learning-loop.md)
- [Prompt Layering and Cache Stability](../concepts/prompt-layering-and-cache-stability.md)
- [CLI to Agent Loop Composition](../syntheses/cli-to-agent-loop-composition.md)
