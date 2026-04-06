# OpenCode Wiki Operations Log

> Append-only record of build, ingest, query, and lint operations.

---

## 2026-04-06 — Wiki Creation

**Operation:** Build
**Pages affected:** schema.md, index.md, log.md
**Sources read:** `opencode/README.md`, `opencode/AGENTS.md`, root `package.json`, package manifests, key source files in `packages/opencode/src`, and selected client package READMEs
**Notes:** Created a dedicated OpenCode sub-wiki so OpenCode appears as a peer knowledge base alongside Claude Code and Deep Agents.

## 2026-04-06 — Initial Ingest

**Operation:** Ingest
**Pages affected:** 28 content pages across `summaries/`, `entities/`, `concepts/`, and `syntheses/`
**Sources read:** `packages/opencode/src/index.ts`, `cli/bootstrap.ts`, `session/index.ts`, `tool/registry.ts`, `provider/provider.ts`, `server/router.ts`, `server/server.ts`, `project/bootstrap.ts`, `project/instance.ts`, `control-plane/workspace.ts`, `plugin/index.ts`, `mcp/index.ts`, `acp/agent.ts`, `storage/storage.ts`, `sync/index.ts`, and client package READMEs
**Notes:** Captured OpenCode as a provider-agnostic, client/server coding-agent monorepo with session orchestration, tool composition, project-scoped instances, remote workspaces, and multi-client surfaces.

## 2026-04-06 — Initial Lint

**Operation:** Lint
**Pages affected:** index.md and all content pages
**Sources read:** wiki pages only
**Notes:** Verified directory structure, page inventory, and cross-reference presence during initial build.

## 2026-04-06 — Depth Normalization

**Operation:** Ingest
**Pages affected:** 28 content pages plus `schema.md`
**Sources read:** Existing wiki pages, peer wiki baseline pages in `docs/`, and representative raw source files across the main runtime packages for OpenCode
**Notes:** Rewrote the wiki in place to align with the shared Claude Code depth standard: deeper summaries, implementation-heavy entity pages, mechanism-oriented concept pages, synthesis pages with explicit boundary handoffs, and denser source evidence.
