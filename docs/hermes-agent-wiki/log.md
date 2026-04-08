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
