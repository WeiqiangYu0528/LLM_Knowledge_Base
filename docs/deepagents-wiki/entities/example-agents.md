# Example Agents

## Overview

The `examples/` tree is not filler. It is a catalog of reference architectures showing how the SDK's abstractions are meant to be applied in practice. Each example emphasizes a different composition of tools, memory, skills, subagents, and execution environments.

Examples are part of the design vocabulary: they show which customization patterns the repo expects users to copy.

In practice, this page should be read as a boundary explanation, not just a file list. Example Agents owns a specific slice of the Deep Agents runtime, but it only makes sense in relation to neighboring systems such as [Graph Factory](graph-factory.md), [Skills System](skills-system.md), [Filesystem First Agent Configuration](../concepts/filesystem-first-agent-configuration.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `examples/README.md` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `examples/async-subagent-server/README.md`, `examples/content-builder-agent/README.md`, `examples/deep_research/README.md` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Catalog of example folders and their intended use cases; Hosted async subagent pattern; Filesystem-configured content agent; Research supervisor with subagents and search |
| Extension or policy points | The main extension or gating surfaces live around `examples/content-builder-agent/README.md`. |
| Persistent effects | Hosted async subagent pattern |

## Architecture

Example Agents is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `examples/README.md`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Example Agents does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `examples/README.md` becomes relevant when catalog of example folders and their intended use cases.
2. `async-subagent-server/README.md` becomes relevant when hosted async subagent pattern.
3. `content-builder-agent/README.md` becomes relevant when filesystem-configured content agent.
4. `deep_research/README.md` becomes relevant when research supervisor with subagents and search.
5. `text-to-sql-agent/README.md` becomes relevant when sQL-oriented planning agent.
6. `ralph_mode/README.md` becomes relevant when fresh-context looping pattern.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around `examples/content-builder-agent/README.md`. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `examples/README.md` | Catalog of example folders and their intended use cases |
| `examples/async-subagent-server/README.md` | Hosted async subagent pattern |
| `examples/content-builder-agent/README.md` | Filesystem-configured content agent |
| `examples/deep_research/README.md` | Research supervisor with subagents and search |
| `examples/text-to-sql-agent/README.md` | SQL-oriented planning agent |
| `examples/ralph_mode/README.md` | Fresh-context looping pattern |
| `examples/nvidia_deep_agent/README.md` | Multi-model GPU-enabled execution architecture |

## See Also

- [Graph Factory](graph-factory.md)
- [Skills System](skills-system.md)
- [Filesystem First Agent Configuration](../concepts/filesystem-first-agent-configuration.md)
- [Evaluation Feedback Loop](../syntheses/evaluation-feedback-loop.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
