# Plugin and Memory Provider System

## Overview

Hermes has two extension families that look similar at first glance but serve different architectural roles:

- the general plugin system in `hermes_cli/plugins.py`
- the repo-bundled memory-provider family in `plugins/memory/`

The general plugin system is open-ended: tools, hooks, and CLI commands can come from user directories, project directories, or Python entry points. Memory providers are narrower and more controlled: they are repo-bundled, one external provider may be active at a time, and they integrate tightly with `MemoryManager`.

## Key Types / Key Concepts

| Type or module | Role |
|------|------|
| `PluginManager` and `PluginContext` in `hermes_cli/plugins.py` | Discovery and registration for general plugins |
| `PluginManifest` | Parsed plugin metadata from `plugin.yaml` |
| `discover_memory_providers()` in `plugins/memory/__init__.py` | Scans repo-bundled memory providers |
| `MemoryProvider` ABC | Contract implemented by external memory providers |
| plugin hooks | Extension callbacks such as `pre_tool_call` and `post_tool_call` |

## Architecture

### General plugin system

General plugins may come from:

1. `~/.hermes/plugins/<name>/`
2. `./.hermes/plugins/<name>/` when project plugins are enabled
3. packages exposing the `hermes_agent.plugins` entry-point group

Plugins register themselves through `register(ctx)` and can contribute tools, hooks, or CLI commands. The plugin manager tracks which tools and hooks came from each plugin and can skip disabled plugins from config.

### Memory-provider family

Memory providers are discovered separately because their runtime contract is different. They implement `MemoryProvider`, integrate with `MemoryManager`, and are limited to one active external provider. That means they behave more like part of the agent runtime than arbitrary plugin add-ons.

## Runtime Behavior

### Hook registration

General plugins can register lifecycle hooks such as:

- `pre_tool_call`
- `post_tool_call`
- `pre_llm_call`
- `post_llm_call`
- `on_session_start`
- `on_session_end`

That gives them observability around the agent loop without forcing the agent core to know about each plugin explicitly.

### Tool and CLI extension

`PluginContext.register_tool()` delegates to the central tool registry, which means plugin-defined tools appear alongside built-in tools. `register_cli_command()` similarly extends the CLI command tree without editing the main parser directly.

### Memory-provider activation

Memory providers use a separate discovery path and often expose their own tools, config schema, and lifecycle hooks. The distinction matters: a provider can influence prompt construction, prefetch, sync, compression, and session-end behavior in ways a normal plugin usually should not.

## Source Files

| File | Purpose |
|------|---------|
| `hermes_cli/plugins.py` | General plugin discovery, manifests, context, hooks, and CLI/tool registration |
| `plugins/memory/__init__.py` | Memory-provider discovery and loading |
| `agent/memory_provider.py` | Provider ABC and lifecycle contract |
| `plugins/memory/*` | Bundled provider implementations |
| `website/docs/developer-guide/memory-provider-plugin.md` | Guide for implementing provider plugins |

## See Also

- [Memory and Learning Loop](memory-and-learning-loop.md)
- [Skills System](skills-system.md)
- [Cross-Session Recall and Memory Provider Pluggability](../concepts/cross-session-recall-and-memory-provider-pluggability.md)
- [Self-Improving Agent Architecture](../concepts/self-improving-agent-architecture.md)
