# Graph Factory

## Overview

The graph factory is the SDK nucleus. `create_deep_agent` in `libs/deepagents/deepagents/graph.py` builds a compiled LangGraph agent that already knows how to plan, manipulate files, delegate work, and manage context. Rather than exposing only a thin agent constructor, Deep Agents encodes product opinion directly into this assembly layer.

The factory is the policy-rich constructor that decides what a Deep Agents runtime is before any CLI or protocol surface gets involved.

In practice, this page should be read as a boundary explanation, not just a file list. Graph Factory owns a specific slice of the Deep Agents runtime, but it only makes sense in relation to neighboring systems such as [Backend System](backend-system.md), [Subagent System](subagent-system.md), [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md). The source-file map below shows where that ownership begins, where it is delegated, and where policy or state crosses into other modules.

## Key Types / Key Concepts

```python
def create_deep_agent(
    model=None,
    tools=None,
    *,
    system_prompt=None,
    middleware=(),
    subagents=None,
    skills=None,
    memory=None,
    response_format=None,
    context_schema=None,
    checkpointer=None,
    store=None,
    backend=None,
    interrupt_on=None,
    debug=False,
    name=None,
    cache=None,
)
```

| Dimension | Notes |
| --- | --- |
| Primary center of gravity | `libs/deepagents/deepagents/graph.py` is the best first file when you need to confirm the runtime shape of this subsystem. |
| Supporting modules | `libs/deepagents/deepagents/__init__.py`, `libs/deepagents/deepagents/_models.py` refine the boundary by handling adjacent concerns rather than duplicating the primary entry point. |
| Runtime concerns | Main `create_deep_agent` assembly logic and default base prompt; Public SDK export surface; Model resolution helpers used during graph construction |
| Extension or policy points | The main extension or gating surfaces live around the neighboring modules listed in the source map. |
| Persistent effects | state changes and runtime handoffs described in the cited files |

## Architecture

Graph Factory is organized around a clear center-of-gravity module and a ring of support files. The primary entry point is `libs/deepagents/deepagents/graph.py`, which anchors the control flow and makes the main decisions about this subsystem. The surrounding modules exist because the subsystem has to balance orchestration with policy, transport, UI, or persistence concerns rather than collapsing all behavior into one file.

Reading the source map from top to bottom usually reconstructs the intended design. Early files define the public or orchestration surface, mid-level files handle execution mechanics, and later files tend to encode policy, adapters, or stateful helpers. That split is deliberate: it keeps the subsystem usable from the rest of the repo while leaving enough internal structure to swap providers, backends, policies, or UI-specific glue without rewriting the core logic.

For implementers, the important question is not only what Graph Factory does, but what it does not own. Neighboring entity and concept pages explain the adjacent responsibilities, especially where configuration, approval, persistence, routing, or remote execution hand off across boundaries. The runtime usually stays coherent because this subsystem keeps its contract narrow even when the source tree around it is large.

## Runtime Behavior

1. `deepagents/graph.py` becomes relevant when main `create_deep_agent` assembly logic and default base prompt.
2. `deepagents/__init__.py` becomes relevant when public SDK export surface.
3. `deepagents/_models.py` becomes relevant when model resolution helpers used during graph construction.

The ordered path above is where most debugging and extension work should start. If behavior differs from expectations, begin with the first stage that decides policy or state, then walk outward into the linked modules. That approach is more reliable than searching by feature name because the repo often composes behavior across several small files rather than one large implementation.

## Variants, Boundaries, and Failure Modes

This subsystem has meaningful boundaries with the pages linked in See Also. Those links are not ornamental: they identify where model configuration, UI concerns, plugin or protocol loading, routing, approvals, or persistence leave the local code path and become shared runtime behavior. When you need to change behavior safely, check those boundaries first.

The main extension and failure surfaces are concentrated around the neighboring modules listed in the source map. Files with names related to config, hooks, trust, auth, providers, remote execution, or policy usually encode the rules that prevent the subsystem from behaving like a standalone utility. That is why repository-level changes often need both an entity-page update here and a concept or synthesis update elsewhere: the code is modular, but the guarantees are cross-cutting.

## Source Files

| File | Purpose |
| --- | --- |
| `libs/deepagents/deepagents/graph.py` | Main `create_deep_agent` assembly logic and default base prompt |
| `libs/deepagents/deepagents/__init__.py` | Public SDK export surface |
| `libs/deepagents/deepagents/_models.py` | Model resolution helpers used during graph construction |

## See Also

- [Backend System](backend-system.md)
- [Subagent System](subagent-system.md)
- [Batteries Included Agent Architecture](../concepts/batteries-included-agent-architecture.md)
- [Agent Customization Surface](../syntheses/agent-customization-surface.md)
- [Sdk To Cli Composition](../syntheses/sdk-to-cli-composition.md)
- [Architecture Overview](../summaries/architecture-overview.md)
- [Codebase Map](../summaries/codebase-map.md)
