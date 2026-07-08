# ADR 0001 — Durability model: event sourcing

**Status:** Accepted

## Context

A long-running AI agent workflow runs for minutes to hours, executes many steps (LLM calls,
tool calls, sub-agents) with real side effects, and must survive a worker crash without
losing completed work or duplicating side effects on retry. The durability model is the
intellectual heart of the platform — every other capability depends on how workflow state
is persisted and recovered.

## Decision

Use **event sourcing** with an external state machine: a workflow's progress is an
append-only log of events in PostgreSQL, and its live state is a deterministic function of
that log. A crashed run is resumed by replaying the log, skipping already-completed steps,
and continuing.

## Alternatives considered

- **State serialization (snapshotting a running process).** Rejected. A running Python
  stack cannot be cleanly snapshotted; such approaches break at serialization boundaries and
  hide non-determinism, making crash recovery unreliable.

## Consequences

- Workflow state is always reconstructable by replaying persisted events (determinism).
- Retries resume rather than restart, so side effects are never duplicated.
- Pause-for-approval falls out naturally as a "waiting for approval" event.
- Workflow code must respect serialization boundaries and keep replay deterministic.
