# Evals System

## Overview

The evals system is a behavioral test harness for Deep Agents. Instead of focusing only on unit-level correctness, it runs agents end to end against real models and scores both correctness and trajectory quality.

The eval harness is architecture pressure, not just reporting; it captures what behaviors the default graph is expected to retain over time.

In practice, this page should be read as a boundary explanation, not just a file list. Evals System owns a specific slice of the Deep Agents runtime, but it only makes sense in relation to neighboring systems such as [Graph Factory](graph-factory.md), [Example Agents](example-agents.md), [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `libs/evals/README.md` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `libs/evals/deepagents_evals/radar.py`, `libs/evals/deepagents_evals/categories.json`, `libs/evals/tests/evals/` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Eval philosophy, test categories, and benchmark coverage; Radar chart generation and category labels; Single source of truth for category names and display labels; End-to-end eval definitions and fixtures |
| Extension or policy points | The main extension or gating surfaces live around the neighboring modules listed in the source map. |
| Persistent effects | state changes and runtime handoffs described in the cited files |

## Architecture

Evals System is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `libs/evals/README.md`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Evals System does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `evals/README.md` becomes relevant when eval philosophy, test categories, and benchmark coverage.
2. `deepagents_evals/radar.py` becomes relevant when radar chart generation and category labels.
3. `deepagents_evals/categories.json` becomes relevant when single source of truth for category names and display labels.
4. `evals/` becomes relevant when end-to-end eval definitions and fixtures.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around the neighboring modules listed in the source map. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `libs/evals/README.md` | Eval philosophy, test categories, and benchmark coverage |
| `libs/evals/deepagents_evals/radar.py` | Radar chart generation and category labels |
| `libs/evals/deepagents_evals/categories.json` | Single source of truth for category names and display labels |
| `libs/evals/tests/evals/` | End-to-end eval definitions and fixtures |

## See Also

- [Graph Factory](graph-factory.md)
- [Example Agents](example-agents.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Evaluation Feedback Loop](../syntheses/evaluation-feedback-loop.md)
- [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
