# Deep Agents Wiki Operations Log

> Append-only record of build, ingest, query, and lint operations.

---

## 2026-04-06 — Wiki Creation

**Operation:** Build
**Pages affected:** schema.md, index.md, log.md
**Sources read:** `deepagents/README.md`, `deepagents/AGENTS.md`, `deepagents/libs/README.md`, package READMEs and key source files across SDK, CLI, ACP, evals, and examples
**Notes:** Created a dedicated Deep Agents sub-wiki inside the broader knowledge base rather than merging this repository into the existing Claude Code architecture wiki.

## 2026-04-06 — Initial Ingest

**Operation:** Ingest
**Pages affected:** 27 content pages across `summaries/`, `entities/`, `concepts/`, and `syntheses/`
**Sources read:** `libs/deepagents/deepagents/graph.py`, backend and middleware modules, `libs/cli/deepagents_cli/*`, `libs/acp/deepagents_acp/server.py`, `libs/evals/README.md`, `libs/evals/deepagents_evals/radar.py`, example READMEs, partner package READMEs
**Notes:** Captured the monorepo structure around the SDK graph factory, CLI runtime, filesystem-first customization model, remote execution surfaces, and evaluation stack.

## 2026-04-06 — Initial Lint

**Operation:** Lint
**Pages affected:** index.md and all content pages
**Sources read:** wiki pages only
**Notes:** Verified directory structure, page inventory, and cross-reference presence during initial build.

## 2026-04-06 — Depth Normalization

**Operation:** Ingest
**Pages affected:** 27 content pages plus `schema.md`
**Sources read:** Existing wiki pages, peer wiki baseline pages in `docs/`, and representative raw source files across the main runtime packages for Deep Agents
**Notes:** Rewrote the wiki in place to align with the shared Claude Code depth standard: deeper summaries, implementation-heavy entity pages, mechanism-oriented concept pages, synthesis pages with explicit boundary handoffs, and denser source evidence.
