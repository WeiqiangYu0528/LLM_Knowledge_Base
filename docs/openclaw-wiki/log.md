# OpenClaw Wiki Log

## 2026-04-06 — Build

**Operation:** Build  
**Pages affected:** `schema.md`, `index.md`, `log.md`, all summary pages, all entity pages, all concept pages, all synthesis pages  
**Sources read:** `README.md`, `package.json`, `src/cli/run-main.ts`, `src/gateway/server.impl.ts`, `src/plugins/loader.ts`, `src/channels/registry.ts`, `src/routing/resolve-route.ts`, `src/node-host/runner.ts`, `src/canvas-host/server.ts`, `src/acp/server.ts`, `src/mcp/channel-server.ts`, plus top-level directory structure under `src/`, `extensions/`, `apps/`, `skills/`, and `docs/`  
**Notes:** Created a dedicated OpenClaw peer wiki focused on the gateway control plane, agent runtime, channel/plugin ecosystem, session/routing model, automation surfaces, and device/native client architecture.

## 2026-04-06 — Depth Normalization

**Operation:** Ingest
**Pages affected:** 32 content pages plus `schema.md`
**Sources read:** Existing wiki pages, peer wiki baseline pages in `docs/`, and representative raw source files across the main runtime packages for OpenClaw
**Notes:** Rewrote the wiki in place to align with the shared Claude Code depth standard: deeper summaries, implementation-heavy entity pages, mechanism-oriented concept pages, synthesis pages with explicit boundary handoffs, and denser source evidence.

## 2026-04-08 — Full Depth Rewrite (All Entities, Concepts, Syntheses)

**Operation:** Ingest
**Pages affected:** All 16 entity pages, all 6 concept pages, all 6 synthesis pages (28 total)
**Sources read:** `src/gateway/server.impl.ts`, `src/gateway/server-methods.ts`, `src/gateway/auth.ts`, `src/gateway/auth-rate-limit.ts`, `src/gateway/device-auth.ts`, `src/channels/registry.ts`, `src/channels/channel-config.ts`, `src/channels/chat-type.ts`, `src/routing/resolve-route.ts`, `src/routing/session-key.ts`, `src/routing/bindings.ts`, `src/sessions/session-key-utils.ts`, `src/sessions/session-lifecycle-events.ts`, `src/agents/agent-scope.ts`, `src/agents/bash-tools.exec-approval-request.ts`, `src/plugins/registry.ts`, `src/plugins/loader.ts`, `src/plugins/discovery.ts`, `src/plugins/runtime.ts`, `src/plugins/config-state.ts`, `src/plugins/api-builder.ts`, `src/plugins/sdk-alias.ts`, `src/plugins/min-host-version.ts`, `src/plugins/path-safety.ts`, `src/cron/service.ts`, `src/cron/schedule.ts`, `src/cron/isolated-agent/run.ts`, `src/cron/delivery.ts`, `src/cron/session-reaper.ts`, `src/node-host/exec-policy.ts`, `src/infra/exec-approvals.ts`, `src/secrets/runtime.ts`, `src/secrets/runtime-gateway-auth-surfaces.ts`, `src/acp/policy.ts`, `src/acp/approval-classifier.ts`
**Notes:** Complete source-grounded rewrite of all content pages. Previous pages were 62-72 lines of structural boilerplate with minimal concrete content. All pages now contain: exact TypeScript types with field-level annotations, concrete function signatures with parameter names, runtime constants (e.g., `CRON_EVAL_CACHE_MAX = 512`, `FAILURE_NOTIFICATION_TIMEOUT_MS = 30000`, `OUTPUT_CAP = 200000`), step-by-step runtime behavior traces, invariants derived from the code, and source evidence tables with specific function/type names. Synthesis pages now trace concrete cross-system call chains rather than listing participating entities. Entity pages now include Key Types sections with actual TypeScript signatures. Concept pages now include Mechanism, Invariants, and Source Evidence with real code references.
