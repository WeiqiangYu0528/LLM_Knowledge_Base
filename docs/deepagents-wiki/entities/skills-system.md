# Skills System

## Overview

The skills system implements progressive disclosure for reusable workflows. Skills are loaded from directories containing `SKILL.md` files with YAML frontmatter, then exposed to the model through middleware rather than hardcoded into every prompt.

Skills are treated as filesystem-defined augmentation that can be moved across projects without repackaging the SDK.

In practice, this page should be read as a boundary explanation, not just a file list. Skills System owns a specific slice of the Deep Agents runtime, but it only makes sense in relation to neighboring systems such as [Memory System](memory-system.md), [Subagent System](subagent-system.md), [Filesystem First Agent Configuration](../concepts/filesystem-first-agent-configuration.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `libs/deepagents/deepagents/middleware/skills.py` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `libs/cli/deepagents_cli/skills/load.py`, `libs/cli/deepagents_cli/skills/commands.py`, `libs/cli/deepagents_cli/built_in_skills/remember/SKILL.md` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Skill loading, validation, state schema, and prompt augmentation; CLI-side skill loading helpers; CLI interfaces for skill-related commands; Example of a built-in skill packaged with the CLI |
| Extension or policy points | The main extension or gating surfaces live around the neighboring modules listed in the source map. |
| Persistent effects | state changes and runtime handoffs described in the cited files |

## Architecture

Skills System is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `libs/deepagents/deepagents/middleware/skills.py`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Skills System does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `middleware/skills.py` becomes relevant when skill loading, validation, state schema, and prompt augmentation.
2. `skills/load.py` becomes relevant when cLI-side skill loading helpers.
3. `skills/commands.py` becomes relevant when cLI interfaces for skill-related commands.
4. `remember/SKILL.md` becomes relevant when example of a built-in skill packaged with the CLI.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around the neighboring modules listed in the source map. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `libs/deepagents/deepagents/middleware/skills.py` | Skill loading, validation, state schema, and prompt augmentation |
| `libs/cli/deepagents_cli/skills/load.py` | CLI-side skill loading helpers |
| `libs/cli/deepagents_cli/skills/commands.py` | CLI interfaces for skill-related commands |
| `libs/cli/deepagents_cli/built_in_skills/remember/SKILL.md` | Example of a built-in skill packaged with the CLI |

## See Also

- [Memory System](memory-system.md)
- [Subagent System](subagent-system.md)
- [Filesystem First Agent Configuration](../concepts/filesystem-first-agent-configuration.md)
- [Agent Customization Surface](../syntheses/agent-customization-surface.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
