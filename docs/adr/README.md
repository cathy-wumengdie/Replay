# Architecture Decision Records

This log records significant design decisions for Conductor, along with the alternatives
that were considered and rejected. Each ADR captures the context, the decision, and the
reasoning at the time it was made.

| # | Title | Status |
|---|-------|--------|
| [0001](0001-durability-model.md) | Durability model: event sourcing | Accepted |
| [0002](0002-queue-technology.md) | Queue technology: Redis → SQS | Accepted |
| [0003](0003-orchestration.md) | Orchestration: AWS ECS | Accepted |

## Format

Each record follows a lightweight template: **Context** (the forces at play), **Decision**
(what was chosen), **Alternatives considered** (and why they were rejected), and
**Consequences**.
