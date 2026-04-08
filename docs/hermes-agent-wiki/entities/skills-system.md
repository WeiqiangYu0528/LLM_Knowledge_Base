# Skills System

## Overview

Skills are Hermes' preferred instruction-layer extension mechanism. Instead of forcing every reusable workflow into Python code, Hermes lets capabilities live as directories containing `SKILL.md` and optional helpers. This makes skills part of runtime behavior, not just documentation: the skill catalog influences what appears in the prompt, which secrets are requested, and which environment variables are passed through to sandboxes.

## Key Types / Key Concepts

| Function or concept | Role |
|------|------|
| `skills_list` / `skill_view` in `tools/skills_tool.py` | Progressive-disclosure API for listing and loading skills |
| `SKILLS_DIR` in `tools/skills_tool.py` | Profile-scoped installed skills directory under `HERMES_HOME` |
| `skills_config.py` | Global and per-platform enable/disable configuration |
| frontmatter metadata | Name, description, platforms, tool requirements, setup requirements |
| bundled vs optional skills | Always-shipped vs explicitly installed official skills |

## Architecture

Hermes supports several skill sources:

- bundled skills from `skills/`
- official optional skills from `optional-skills/`
- installed copies inside `HERMES_HOME/skills`

At runtime, Hermes treats the `HERMES_HOME/skills` tree as the working catalog. That lets bundled skills, user-installed skills, and agent-edited skills coexist without mutating the source repository.

The tool layer then exposes two main operations:

- list skills cheaply using metadata only
- load a specific skill or reference file on demand

This "progressive disclosure" design keeps the system prompt smaller than if every skill's full instructions were injected all the time.

## Runtime Behavior

### Frontmatter gating

Skill frontmatter can control:

- platform compatibility
- required tools or toolsets
- fallback behavior when certain tools exist
- required environment variables
- non-secret config keys

That means skills are not merely static text. Their metadata affects discovery and whether they should even appear for a given session.

### Setup and passthrough

When a skill declares required environment variables, Hermes can prompt for them during local CLI setup and pass them through into terminal or code-execution sandboxes. Messaging surfaces do not collect secrets in-band; they surface guidance instead. This is an important separation between local secure setup and remote runtime use.

### User-facing management

`hermes skills` and `skills_config.py` give the user platform-aware enable/disable control. A skill can be globally disabled or turned off only for Telegram, Slack, or another surface, which keeps the prompt surface tailored to each environment.

## Source Files

| File | Purpose |
|------|---------|
| `tools/skills_tool.py` | Skill discovery, frontmatter parsing, readiness and view tools |
| `hermes_cli/skills_config.py` | Global and per-platform skill enable/disable controls |
| `website/docs/developer-guide/creating-skills.md` | Maintainer guide for skill structure and metadata |
| `skills/`, `optional-skills/` | Bundled and optional official skill catalogs |

## See Also

- [Config and Profile System](config-and-profile-system.md)
- [Memory and Learning Loop](memory-and-learning-loop.md)
- [Toolset-Based Capability Governance](../concepts/toolset-based-capability-governance.md)
- [Self-Improving Agent Architecture](../concepts/self-improving-agent-architecture.md)
