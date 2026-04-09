# Skills System

## Overview

Hermes uses skills as reusable instruction bundles. A skill is not just a note on disk, and it is not just another Python tool. It is a directory-centered workflow package whose main `SKILL.md` file can be discovered, surfaced in prompts, invoked through slash commands, loaded on demand through the skills tools, and even edited by the agent after a task succeeds.

That makes the skills subsystem a middle layer between static documentation and executable tooling. A tool like `terminal` or `web_search` does work directly. A skill tells the model how to use the capabilities it already has, when a particular workflow or domain pattern is relevant. Hermes uses that layer to keep complex procedural knowledge out of the always-on system prompt while still making it available when needed.

The short mental model is:

1. Hermes builds a skill catalog from `~/.hermes/skills/` plus optional external skill directories.
2. It exposes that catalog in three different ways depending on the runtime path.
3. Metadata in `SKILL.md` frontmatter controls visibility, setup guidance, and some runtime side effects.
4. The model loads full skill content only when it explicitly needs that skill.

The three exposure paths are the most important distinction on this page:

- **Prompt index exposure**: Hermes can show the model a compact skill index in the system prompt.
- **Slash-command exposure**: shells such as the CLI and gateway can turn installed skills into `/skill-name` commands.
- **Tool exposure**: the model can call `skills_list`, `skill_view`, and `skill_manage` to discover, load, or edit skills directly.

Those paths overlap, but they are not the same mechanism. Most of the confusion in this subsystem disappears once they are separated.

## Key Interfaces and Configs

The core interfaces are small, but each one sits at a different stage of the runtime.

| Anchor | Why it matters |
| --- | --- |
| `build_skills_system_prompt()` in `hermes-agent/agent/prompt_builder.py` | Builds the compact skills index that can be injected into the model's system prompt. |
| `scan_skill_commands()` and `build_skill_invocation_message()` in `hermes-agent/agent/skill_commands.py` | Turn installed skills into slash commands and load the full skill content when a user invokes one. |
| `build_preloaded_skills_prompt()` in `hermes-agent/agent/skill_commands.py` | Loads specific skills into the CLI session prompt at startup when the user explicitly preloads them. |
| `_find_all_skills()`, `skills_list()`, and `skill_view()` in `hermes-agent/tools/skills_tool.py` | Implement progressive disclosure: cheap metadata listing first, full skill content or linked files only on demand. |
| `skill_manage()` in `hermes-agent/tools/skill_manager_tool.py` | Lets the agent create, patch, edit, and delete skills as procedural knowledge artifacts. |
| `sync_skills()` in `hermes-agent/tools/skills_sync.py` | Seeds bundled repo skills into `~/.hermes/skills/` and updates them conservatively without overwriting user changes. |
| `get_disabled_skill_names()` and `get_external_skills_dirs()` in `hermes-agent/agent/skill_utils.py` | Define two important catalog rules: what is disabled and which extra directories count as part of the catalog. |
| `skills_config.py` | Stores global and per-platform disabled-skill settings in `config.yaml`. |
| `skills_guard.py` | Security scanner and trust policy for externally sourced or agent-created skills. |

There are also two important runtime roots:

| Path or config | Runtime meaning |
| --- | --- |
| `~/.hermes/skills/` | The main working catalog and single writable source of truth. Bundled skills are copied here, hub-installed skills land here, and agent-created skills are written here. |
| `skills.external_dirs` in `config.yaml` | Optional read-only catalog extensions scanned after the local directory. |
| `skills.disabled` / `skills.platform_disabled` in `config.yaml` | Global and per-platform skill visibility controls. |
| `metadata.hermes.*` frontmatter in `SKILL.md` | Conditional activation, tags, related skills, config declarations, and other Hermes-specific behavior. |

## Architecture

Hermes splits the skills subsystem into four responsibilities.

| Responsibility | Owns | Does not own |
| --- | --- | --- |
| Catalog discovery | Finding installed skills, grouping them by category, handling local vs external directories, and applying disabled/platform filters | Actually loading full skill bodies into every prompt |
| Exposure paths | Prompt index generation, slash-command registration, and tool-based discovery/loading | Executing the workflows described by the skill |
| Setup and metadata interpretation | Required env vars, config injection, platform matching, fallback/requirement conditions, linked-file discovery | Toolset visibility rules themselves |
| Skill lifecycle management | Seeding bundled skills, hub install/update/audit, agent-created edits, security scanning | General memory storage or declarative user facts |

This architecture is why skills sit beside tools instead of inside them. Tools define capabilities Hermes can execute. Skills define procedures for how the model should use those capabilities in particular situations.

That boundary is especially important around memory and learning. Skills are one form of procedural memory, but they are not the same subsystem as `memory`, `session_search`, or long-term memory providers. Memory stores facts, preferences, prior sessions, or learned observations. Skills store reusable workflows in document-plus-files form. Hermes connects them socially, by encouraging the agent to save successful workflows as skills, but the storage model and runtime path are different.

## Runtime Behavior

Before the step-by-step flow, it helps to name the main skill sources Hermes works with:

| Skill source | How it enters the runtime catalog | Normal location once available |
| --- | --- | --- |
| Bundled built-in skills | Shipped in the repo under `skills/` and seeded into the user catalog by `sync_skills()` | `~/.hermes/skills/` |
| Official optional skills | Browsed and installed through the Skills Hub from the repo's `optional-skills/` catalog | `~/.hermes/skills/` |
| User-created or agent-created skills | Written directly through `skill_manage()` or manual edits | `~/.hermes/skills/` |
| Project or team skill directories | Added through `skills.external_dirs` and scanned read-only after the local catalog | Original external path |

### 1. Hermes builds the catalog from the local skills directory first

The main working catalog is `~/.hermes/skills/`. `tools/skills_tool.py` treats this directory as the primary source of truth, and multiple subsystems are built around that assumption.

On the CLI path, `hermes_cli/main.py` calls `tools.skills_sync.sync_skills()` during startup. That sync step copies bundled repository skills into `~/.hermes/skills/` and records their origin hashes in `.bundled_manifest`. The update logic is conservative:

- new bundled skills are copied in
- unchanged user copies can be updated when the bundled version changes
- user-modified copies are left alone
- user-deleted skills are not silently recreated

This is an important design choice. Hermes does not treat the repo's `skills/` directory as the live runtime catalog. It treats the repo as a seed source and `~/.hermes/skills/` as the mutable working catalog.

Official optional skills follow a different intake path. They live in the repo's `optional-skills/` catalog, but they do not arrive automatically through `sync_skills()`. Instead, the Skills Hub exposes them as installable official skills, after which they also land in `~/.hermes/skills/` and behave like any other installed skill.

### 2. External skill directories extend the catalog, but do not replace it

`agent.skill_utils.get_external_skills_dirs()` reads `skills.external_dirs` from `config.yaml`. Those directories are scanned after the local skills directory.

The precedence rule is simple and important:

- local `~/.hermes/skills/` entries win
- external directories are scanned after local
- duplicate names from external dirs are ignored when a local skill with the same name already exists

That rule appears in more than one place. `_find_all_skills()` in `tools/skills_tool.py`, `scan_skill_commands()` in `agent/skill_commands.py`, and `build_skills_system_prompt()` in `agent/prompt_builder.py` all scan local first and treat it as the precedence winner.

So Hermes supports shared team catalogs, but local overrides remain straightforward.

### 3. Discovery is metadata-first, not full-content-first

The skills subsystem is built around progressive disclosure.

`skills_list()` returns a small metadata view: name, description, and category. `skill_view()` loads the full `SKILL.md` only when the model or shell actually asks for it. If the skill has `references/`, `templates/`, `scripts/`, or `assets/`, those files are listed and then loaded one at a time through `skill_view(name, file_path=...)`.

That is not just a token optimization trick. It is the reason Hermes can expose a large skill catalog without injecting every workflow document into every conversation. The prompt index can stay compact, and the model can still drill down when a skill is genuinely relevant.

The directory layout shapes this behavior directly:

| Layout piece | Runtime effect |
| --- | --- |
| `SKILL.md` | Main instruction body and frontmatter metadata |
| `references/` | Extra docs the model can load after seeing the main skill |
| `templates/` | Reusable output or file templates |
| `scripts/` | Helper scripts the skill may direct the model to inspect or use |
| `assets/` | Supplementary files surfaced as linked files |
| `DESCRIPTION.md` in category dirs | Category-level descriptions used in prompt indexing and category discovery |

Because Hermes loads linked files separately, the skill directory layout is not cosmetic. It decides how much of the skill the model sees up front and how much stays behind an explicit read step.

### 4. The prompt index is one exposure path, not the skill itself

`agent.prompt_builder.build_skills_system_prompt()` builds a compact skills index for the system prompt. That index is not the same as loading a skill. It is a category-organized list of available skills and short descriptions, plus instructions telling the model to call `skill_view(name)` if a listed skill clearly matches the task.

This path is filtered before the model ever sees it:

- incompatible platforms are excluded
- disabled skills are excluded
- conditional activation metadata is applied
- local skills take precedence over external duplicates

The conditional activation rules come from `metadata.hermes` frontmatter and are interpreted by `_skill_should_show(...)` in `prompt_builder.py`:

- `fallback_for_toolsets` and `fallback_for_tools` hide a skill when the primary tool or toolset is available
- `requires_toolsets` and `requires_tools` hide a skill when required tools are missing

This is where skills relate to tool governance without becoming tool governance. The tool registry still decides which tools exist in the session. The skills system looks at that tool availability and filters which skills should be shown as relevant.

That distinction matters. A skill cannot make a missing tool appear. It can only say "show me when that tool is missing" or "show me only when that tool exists."

### 5. Slash commands are a second exposure path

`agent.skill_commands.scan_skill_commands()` turns installed skills into slash commands by scanning the same directories and frontmatter. The command key is normalized into a clean slug such as `/github-pr-workflow`, while still tolerating underscore variants in user input.

This path has its own runtime behavior:

- it respects platform compatibility
- it respects disabled-skill config
- it uses local-first precedence across local and external directories
- it loads the full skill body only when the user actually invokes the slash command

When a slash command is invoked, `build_skill_invocation_message()` loads the skill via `skill_view()`, wraps it in an activation note, injects resolved config values when the skill declares them, and includes setup hints or linked-file guidance as needed.

That means slash commands are not a separate skills storage system. They are a shell-facing convenience layer over the same underlying catalog and loader.

`/plan` is a useful comparison point here. It lives in the same shared command helper module, but it is a prompt-style built-in mode rather than an installed skill discovered from the catalog. So the slash-command surface contains both discovered skills and a few adjacent built-ins, but they are not all sourced the same way.

### 6. Session preloading is a third exposure path

The CLI also supports explicit preloading through `build_preloaded_skills_prompt()`. When the user starts a session with `--skills`, Hermes resolves those identifiers immediately, loads the full skill content, and appends it to the session's system prompt before normal conversation begins.

This is different from the normal prompt index path:

- the index path tells the model what skills exist and asks it to load them when relevant
- the preload path makes specific skills active from the start of the session

That distinction is worth making explicit because they affect behavior differently. A preloaded skill behaves like session-wide guidance. A merely indexed skill still has to be selected and loaded.

### 7. `skill_view()` is where metadata starts affecting runtime behavior

Loading a skill does more than read markdown off disk.

`skill_view()` parses frontmatter and can:

- reject skills that do not match the current OS platform
- block viewing of user-disabled skills
- surface setup-needed state for missing environment variables or credential files
- register available env vars for passthrough into `terminal` and `execute_code`
- inject or expose linked files for follow-up loading
- expose tags, related skills, and setup hints

This is the point where the skill's metadata starts shaping the runtime, not just its discoverability.

The setup behavior is especially important. If a skill declares required environment variables, Hermes does not simply hide the skill. On local interactive surfaces it can prompt securely when the skill is loaded. On messaging surfaces it does not collect secrets in-band; it returns guidance instead. Either way, the skill remains discoverable, but its readiness state changes.

That is a good example of why skills are more than docs and less than tools: their metadata affects runtime use, but they still operate by guiding the model rather than directly executing capability logic.

### 8. `skill_manage()` makes skills part of learning, but not part of memory storage

`tools/skill_manager_tool.py` lets the agent create, patch, edit, and delete skills under `~/.hermes/skills/`, plus manage supporting files. The tool validates names and frontmatter, constrains writable subdirectories, uses atomic writes, and runs the same security scanner used for external skill installs.

This is where Hermes turns successful procedures into reusable artifacts. The prompt layer reinforces that pattern too: `prompt_builder.py` tells the agent to offer saving difficult or iterative workflows as skills, and to patch skills that were missing steps or had wrong commands.

This is why the subsystem is adjacent to memory and learning:

- **memory tools** capture facts, preferences, or session-derived knowledge
- **skills** capture reusable "how to do this" procedures

They are connected in intent, but they are different storage and execution paths. A memory item is not automatically a skill, and a skill is not just a memory blob.

### 9. Installation and trust live around the catalog

Hermes also has a wider lifecycle around skills through the Skills Hub and guards. `hermes_cli/skills_hub.py` gives the user browse/search/install/update/audit flows across official optional skills and external registries. `tools/skills_guard.py` scans externally sourced and agent-created skills for prompt injection, exfiltration, destructive commands, persistence behavior, and similar patterns.

This matters for the page because it shows another boundary:

- discovery and loading assume the skill is already in the catalog
- hub install, quarantine, audit, and trust policy decide how a skill enters the catalog safely

The security layer surrounds the skills catalog. It is not part of normal prompt injection or slash-command routing once the skill is already installed.

## Source Files

| File | Why it is an anchor |
| --- | --- |
| `hermes-agent/agent/prompt_builder.py` | Builds the compact skills index for the system prompt and applies conditional activation rules based on available tools and toolsets. |
| `hermes-agent/agent/skill_commands.py` | Shared slash-command discovery and invocation helpers, plus CLI preloaded-skill prompt assembly. |
| `hermes-agent/agent/skill_utils.py` | Lightweight metadata helpers for frontmatter parsing, disabled-skill config, external dirs, and conditional activation fields. |
| `hermes-agent/tools/skills_tool.py` | Core catalog discovery, progressive-disclosure APIs, setup/readiness handling, linked-file loading, and skill metadata interpretation. |
| `hermes-agent/tools/skill_manager_tool.py` | Agent-managed creation and editing of skills as reusable procedural knowledge. |
| `hermes-agent/tools/skills_sync.py` | Seeds and updates bundled skills into the live `~/.hermes/skills/` catalog without trampling user changes. |
| `hermes-agent/tools/skills_guard.py` | Security scanner and trust policy for hub-installed or agent-created skills. |
| `hermes-agent/hermes_cli/skills_hub.py` | User-facing browse/search/install/update/audit flows around the skill catalog. |
| `hermes-agent/hermes_cli/skills_config.py` | Global and per-platform disabled-skill configuration. |
| `hermes-agent/website/docs/user-guide/features/skills.md` | Maintainer and user-facing documentation for the same progressive-disclosure, external-dir, setup, and hub behaviors. |

Taken together, these files show a consistent ownership model: `skills_tool.py` owns the live catalog interface, `prompt_builder.py` and `skill_commands.py` decide how that catalog is surfaced, and the hub/sync/guard tools manage how skills safely arrive and evolve.

## See Also

- [Tool Registry and Dispatch](tool-registry-and-dispatch.md)
- [Config and Profile System](config-and-profile-system.md)
- [Memory and Learning Loop](memory-and-learning-loop.md)
- [Plugin and Memory Provider System](plugin-and-memory-provider-system.md)
- [Toolset-Based Capability Governance](../concepts/toolset-based-capability-governance.md)
- [Self-Improving Agent Architecture](../concepts/self-improving-agent-architecture.md)
