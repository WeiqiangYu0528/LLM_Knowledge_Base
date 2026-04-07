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

## 2026-04-07 — Full Source-Grounded Rewrite

**Operation:** Ingest
**Pages affected:** All 13 entity pages, all 6 concept pages, all 5 synthesis pages
**Sources read:**
- `libs/deepagents/deepagents/graph.py` — full `create_deep_agent()` signature, `BASE_AGENT_PROMPT`, middleware assembly
- `libs/deepagents/deepagents/middleware/` — subagents, async_subagents, filesystem, skills, memory, summarization, patch_tool_calls
- `libs/deepagents/deepagents/backends/` — protocol, local_shell, sandbox, filesystem, store, composite, langsmith
- `libs/deepagents/deepagents/_models.py`
- `libs/cli/deepagents_cli/main.py`, `agent.py`, `app.py`, `config.py`, `mcp_tools.py`, `ask_user.py`, `local_context.py`, `command_registry.py`, `built_in_skills/`
- `libs/acp/` — ACP server implementation
- `libs/evals/` — TrajectoryScorer, test categories, Harbor integration
- `libs/partners/` — Daytona, Modal, Runloop, QuickJS
- `examples/` — deep_research, content-builder-agent, text-to-sql-agent, ralph_mode, nvidia_deep_agent, async-subagent-server
- `libs/deepagents/pyproject.toml`, `libs/cli/pyproject.toml`, `libs/acp/pyproject.toml`
**Notes:** All pages replaced generic template boilerplate with content derived directly from source file reading. Entity pages now include typed signatures, real class/function names, and code snippets. Concept pages explain mechanisms with source evidence. Synthesis pages trace concrete call chains and data flows with real identifiers. Page lengths grew from ~65 lines (boilerplate) to 100–260 lines (substantive) per page.
