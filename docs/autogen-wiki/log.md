# AutoGen Wiki Log

## 2026-04-06 - Initial Build

- Created the initial `autogen-wiki` peer wiki under `docs/`
- Scoped the wiki around four repository layers:
  - Python framework stack: `autogen-core`, `autogen-agentchat`, `autogen-ext`
  - App and tooling surfaces: `autogen-studio`, `agbench`, `magentic-one-cli`
  - Shared runtime contracts: `docs/design/`, `protos/`
  - .NET ecosystem: legacy `AutoGen.*` and new `Microsoft.AutoGen.*`
- Identified hub pages:
  - `summaries/architecture-overview.md`
  - `entities/python-core-runtime.md`
  - `entities/python-agentchat.md`
  - `entities/python-extensions.md`
  - `entities/distributed-runtime-and-worker-system.md`
  - `entities/autogen-studio.md`
  - `entities/dotnet-runtime-stack.md`
  - `concepts/event-driven-agent-programming-model.md`
  - `concepts/local-to-distributed-runtime-scaling.md`
  - `syntheses/core-to-agentchat-to-extension-composition.md`
  - `syntheses/distributed-agent-worker-lifecycle.md`
  - `syntheses/python-and-dotnet-ecosystem-relationship.md`
- Diagram plan for the first pass:
  - ecosystem architecture map in `summaries/architecture-overview.md`
  - distributed worker lifecycle diagram in `syntheses/distributed-agent-worker-lifecycle.md`
- Applied the shared [Claude Code Depth Standard](../depth-standard.md) as the baseline for page contracts, evidence density, and lint expectations.

## See Also

- [AutoGen Wiki](index.md)
- [AutoGen Wiki Schema](schema.md)
- [Architecture Overview](summaries/architecture-overview.md)
