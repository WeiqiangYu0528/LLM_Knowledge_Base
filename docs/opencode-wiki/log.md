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

## 2026-04-09 — Full Depth Rewrite (All Entities, Concepts, Syntheses)

**Operation:** Ingest
**Pages affected:** All 14 entity pages, all 6 concept pages, all 5 synthesis pages (25 total)
**Sources read:** `packages/opencode/src/index.ts`, `provider/provider.ts`, `provider/models.ts`, `provider/transform.ts`, `provider/schema.ts`, `session/index.ts`, `session/message-v2.ts`, `session/prompt.ts`, `permission/index.ts`, `permission/evaluate.ts`, `project/instance.ts`, `project/project.ts`, `project/state.ts`, `plugin/index.ts`, `plugin/loader.ts`, `tool/registry.ts`, `tool/tool.ts`, `tool/bash.ts`, `server/server.ts`, `server/router.ts`, `server/routes/*`, `server/middleware.ts`, `server/mdns.ts`, `server/projectors.ts`, `control-plane/workspace.ts`, `sync/index.ts`, `storage/storage.ts`, `storage/db.ts`, `storage/json-migration.ts`, `mcp/index.ts`, `acp/agent.ts`, `lsp/*`, `cli/cmd/serve.ts`, `cli/cmd/web.ts`, `cli/cmd/tui/attach.ts`, `cli/ui.ts`, `installation/index.ts`, plus `packages/desktop-electron/`, `packages/desktop/`, `packages/slack/`, `packages/sdk/js/`, `packages/enterprise/`, `packages/extensions/`
**Notes:** Complete source-grounded rewrite of all 25 content pages. Previous pages were 60-76 lines of structural boilerplate. All entity pages now contain exact TypeScript type signatures (`Permission.Rule`, `Permission.Request`, `InstanceContext`, `Session.Info`, `ModelsDev.Model`), concrete function names (`evalRule()`, `Instance.provide()`, `wrapSSE()`, `shouldUseCopilotResponsesApi()`, `fuzzysort`), runtime constants, and step-by-step execution traces. Concept pages now have Mechanism, Invariants, and Source Evidence sections with code-level detail. All 5 synthesis pages were rewritten to 219-362 lines each, with `## Step-by-Step Flow`, `## Data at Each Boundary`, `## Failure Points`, and `## Source Evidence` sections containing specific function and type names. Key coverage added: all 22+ provider SDK factories, full permission gate sequence (`Permission.Event.Asked`/`Permission.Event.Replied`), `wrapSSE()` SSE stall timeout, `ProviderTransform` post-processing, `MDNS` bonjour-service discovery, `WorkspaceRouterMiddleware` routing logic, desktop Electron sidecar management, Tauri shell, Slack Bolt Socket Mode, JavaScript SDK `createOpencodeClient()`, Zed extension bootstrap, `Heap.start()` memory watchdog, and context compaction `revert` field recovery.
