# Hermes Agent Wiki Operations Log

> Append-only record of build, ingest, query, and lint operations.

---

## 2026-04-08 — Wiki Creation

**Operation:** Build  
**Pages affected:** `schema.md`, `index.md`, `log.md`  
**Sources read:** `hermes-agent/README.md`, `hermes-agent/pyproject.toml`, directory structure, and the approved Hermes wiki plan  
**Notes:** Created Hermes Agent as a peer sub-wiki inside the shared MkDocs knowledge base rather than merging it into an existing wiki.

## 2026-04-08 — Initial Source-Grounded Ingest

**Operation:** Build  
**Pages affected:** All Hermes summary, entity, concept, and synthesis pages  
**Sources read:** `run_agent.py`, `model_tools.py`, `hermes_state.py`, `hermes_cli/*.py`, `gateway/*.py`, `tools/*.py`, `agent/*.py`, `cron/*.py`, `acp_adapter/*.py`, `plugins/memory/*`, `batch_runner.py`, `environments/*`, and `website/docs/developer-guide/*`  
**Notes:** Built the Hermes wiki around the runtime spine of `AIAgent`, prompt assembly, provider resolution, tool dispatch, session persistence, gateway routing, memory-provider plugins, cron automation, and research surfaces.

## 2026-04-08 — Initial Lint

**Operation:** Lint  
**Pages affected:** `index.md` and all Hermes content pages  
**Sources read:** wiki pages plus MkDocs navigation configuration  
**Notes:** Verified page inventory, cross-link structure, and site integration for the new Hermes peer wiki.

## 2026-04-08 — Diagram Pass

**Operation:** Build  
**Pages affected:** `summaries/architecture-overview.md`, `syntheses/gateway-message-to-agent-reply-flow.md`, and `assets/graphs/*`  
**Sources read:** Hermes wiki pages plus the underlying runtime sources referenced by those pages  
**Notes:** Added two first-pass Excalidraw diagrams: a system architecture map and a gateway message-flow map. Both were rendered to PNG and embedded into the relevant pages.

## 2026-04-09 — Pilot Rewrite For Readability And Depth

**Operation:** Ingest  
**Pages affected:** `summaries/architecture-overview.md`, `entities/agent-loop-runtime.md`, `entities/gateway-runtime.md`, `entities/tool-registry-and-dispatch.md`, `index.md`, `log.md`  
**Sources read:** `run_agent.py`, `agent/prompt_builder.py`, `agent/context_compressor.py`, `hermes_state.py`, `gateway/run.py`, `gateway/session.py`, `gateway/delivery.py`, `gateway/pairing.py`, `tools/registry.py`, `model_tools.py`, `tools/approval.py`, `tools/terminal_tool.py`, `website/docs/developer-guide/architecture.md`, `website/docs/developer-guide/agent-loop.md`, `website/docs/developer-guide/gateway-internals.md`, `website/docs/developer-guide/tools-runtime.md`  
**Hub pages:** `summaries/architecture-overview.md`, `entities/agent-loop-runtime.md`, `entities/gateway-runtime.md`, `entities/tool-registry-and-dispatch.md`  
**Diagrams:** Reused the existing architecture diagram in `summaries/architecture-overview.md`; no new diagrams added in this pass  
**Notes:** Rewrote the Hermes pilot hub pages into a mixed tutorial/reference style to improve human readability, runtime explanation, and stylistic consistency with the stronger Claude Code hub pages. The index now includes orientation text and recommended reading paths. Final build verification passed with only the older pre-existing warnings elsewhere in the broader knowledge base.
