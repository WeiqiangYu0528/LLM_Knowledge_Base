# Terminal and Execution Environments

## Overview

Hermes' terminal and sandbox story is deliberately abstracted behind a common environment interface. The terminal tool should be able to run commands in a local shell, a Docker container, an SSH host, a Modal or Daytona sandbox, or a Singularity environment without the model having to learn a different tool each time. That abstraction lives in `tools/environments/`, with `BaseEnvironment` providing the common contract.

## Key Types / Key Concepts

| Type or backend | Role |
|------|------|
| `BaseEnvironment` in `tools/environments/base.py` | Common execution and cleanup interface for backends |
| `local.py` | Local persistent shell execution |
| `docker.py` | Containerized terminal backend |
| `ssh.py` | Remote shell backend over SSH |
| `modal.py`, `managed_modal.py` | Remote sandbox backends on Modal |
| `daytona.py` | Daytona-backed remote execution |
| `singularity.py` | Singularity-based sandbox execution |

## Architecture

The environment layer exists so the terminal tool can stay stable while runtime placement changes. `BaseEnvironment` owns:

- a unified `execute(...)` interface
- cleanup and shutdown semantics
- shared helpers for subprocess setup, timeouts, and sudo transformation
- a host-side sandbox root under `HERMES_HOME`

Backend modules then decide how to realize that contract. Some use persistent shells, some use one-shot subprocesses, and some proxy into remote services. Hermes therefore treats "terminal execution" as a portable capability rather than a specific local process strategy.

## Runtime Behavior

### Backend selection

The active backend is usually chosen from environment or config, typically through `TERMINAL_ENV` and related settings. The rest of the runtime does not care whether the command runs locally or remotely so long as the backend can satisfy the common interface and return normalized output.

### Shared safety and convenience behavior

The base class centralizes several behaviors that would otherwise drift across backends:

- command preparation, including sudo password handling
- standard timeout result shape
- consistent stdout/stderr capture
- sandbox-root creation in `HERMES_HOME`

This is part of the reason Hermes can reuse terminal capabilities across CLI, gateway, cron, and research surfaces without rewriting every execution path.

### Persistent vs one-shot execution

Not all backends behave the same. Some keep persistent shell state for better session continuity; others naturally execute one-shot commands or remote jobs. Hermes exposes both patterns through the backend interface so the tool layer can preserve context when appropriate but still avoid deadlocks or contention when a one-shot path is safer.

## Source Files

| File | Purpose |
|------|---------|
| `tools/environments/base.py` | Common environment contract and helpers |
| `tools/environments/local.py` | Local persistent shell backend |
| `tools/environments/docker.py` | Docker-backed command execution |
| `tools/environments/ssh.py` | Remote shell backend |
| `tools/environments/modal.py`, `managed_modal.py`, `daytona.py`, `singularity.py` | Hosted and sandboxed execution backends |
| `website/docs/developer-guide/environments.md` | How the same environment ideas surface in the research stack |

## See Also

- [Tool Registry and Dispatch](tool-registry-and-dispatch.md)
- [Environment Abstraction for Agent Execution](../concepts/environment-abstraction-for-agent-execution.md)
- [Tool Call Execution and Approval Pipeline](../syntheses/tool-call-execution-and-approval-pipeline.md)
- [Research and Batch Surfaces](research-and-batch-surfaces.md)
