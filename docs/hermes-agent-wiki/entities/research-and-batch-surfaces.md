# Research and Batch Surfaces

## Overview

Hermes is not only an end-user agent shell. The repository also contains infrastructure for batch rollouts, benchmark execution, RL training environments, and data generation. These surfaces matter because they reuse Hermes' tool-calling runtime and environment abstractions in non-interactive settings, making the repo both a product and an experimentation platform.

## Key Types / Key Concepts

| Type or module | Role |
|------|------|
| `batch_runner.py` | Parallel dataset-driven agent execution with checkpointing and trajectory capture |
| `HermesAgentBaseEnv` in `environments/hermes_base_env.py` | Base class that binds Hermes tools and runtime to Atropos environments |
| `HermesAgentLoop` in `environments/agent_loop.py` | Reusable multi-turn loop for training and evaluation |
| `ToolContext` in `environments/tool_context.py` | Gives evaluators access to the same sandbox state used by the rollout |
| tool-call parsers | Convert raw model output into structured tool calls in some training/eval phases |

## Architecture

The research stack reuses Hermes in two directions:

- `batch_runner.py` launches many ordinary Hermes runs over datasets and captures trajectories.
- `environments/` embeds Hermes tool-calling behavior inside evaluation or RL environments, often under the Atropos framework.

This means the repo has a second "client family" beyond CLI/gateway/ACP: experimentation and benchmarking surfaces that want Hermes' tool use and sandbox logic but not its interactive shell UX.

## Runtime Behavior

### Batch runner

`batch_runner.py` loads datasets, distributes work across processes, runs `AIAgent` over each prompt, normalizes tool stats, and stores trajectories. It is oriented toward throughput, resumability, and comparable output schemas rather than live user interaction.

### Environments and evaluation

`HermesAgentBaseEnv` and related environment classes bind Hermes tools to benchmark or RL tasks. `ToolContext` is especially important because it lets reward functions inspect the same terminal or filesystem state that the model manipulated during the rollout. That keeps evaluation tied to concrete agent effects rather than just final text.

### Parser-backed tool extraction

Some phases of the research stack rely on parser modules under `environments/tool_call_parsers/` to recover tool calls from raw model output. This is distinct from the product runtime, where providers may already return structured tool calls.

## Source Files

| File | Purpose |
|------|---------|
| `batch_runner.py` | Batch rollouts, checkpoints, trajectory output |
| `environments/hermes_base_env.py` | Shared environment base class |
| `environments/agent_loop.py` | Reusable multi-turn loop for research runs |
| `environments/tool_context.py` | Post-rollout access to sandbox state |
| `environments/tool_call_parsers/*` | Parser family for raw output to tool-call conversion |
| `website/docs/developer-guide/environments.md` | Maintainer guide to RL, benchmarks, and data generation |

## See Also

- [Terminal and Execution Environments](terminal-and-execution-environments.md)
- [Agent Loop Runtime](agent-loop-runtime.md)
- [Environment Abstraction for Agent Execution](../concepts/environment-abstraction-for-agent-execution.md)
- [Cron System](cron-system.md)
