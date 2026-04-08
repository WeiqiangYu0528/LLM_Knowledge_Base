# Environment Abstraction for Agent Execution

## Overview

Hermes wants one agent to be able to act in many execution environments. The model should not need to know whether a terminal command runs on the local machine, in a Docker container, or through a hosted backend. That portability is achieved by abstracting execution behind environment backends and by reusing the same abstraction in the research stack.

## Mechanism

At the product-runtime layer:

1. The terminal tool resolves the active backend.
2. The backend implements `execute(...)` and cleanup on top of `BaseEnvironment`.
3. Hermes normalizes outputs back into tool results.

At the research layer:

1. environments configure a Hermes-compatible terminal backend
2. rollouts call Hermes tools
3. evaluators inspect the resulting sandbox state through `ToolContext`

This means environment abstraction is not just a terminal convenience. It is a shared contract across user-facing and benchmarking surfaces.

## Involved Entities

- [Terminal and Execution Environments](../entities/terminal-and-execution-environments.md)
- [Tool Registry and Dispatch](../entities/tool-registry-and-dispatch.md)
- [Research and Batch Surfaces](../entities/research-and-batch-surfaces.md)

## Source Evidence

- `tools/environments/base.py` — common backend contract
- `tools/environments/local.py`, `docker.py`, `ssh.py`, `modal.py`, `daytona.py`, `singularity.py` — concrete placement strategies
- `environments/hermes_base_env.py` — reuse of Hermes execution backends in the research stack
- `environments/tool_context.py` — sandbox-state access for evaluation

## See Also

- [Terminal and Execution Environments](../entities/terminal-and-execution-environments.md)
- [Research and Batch Surfaces](../entities/research-and-batch-surfaces.md)
- [Tool Call Execution and Approval Pipeline](../syntheses/tool-call-execution-and-approval-pipeline.md)
- [Agent Loop Runtime](../entities/agent-loop-runtime.md)
