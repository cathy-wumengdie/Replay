# Conductor

**A durable execution platform for long-running AI agent workflows.**

An event-sourced runtime for durable, long-running AI agent workflows — Temporal-style
durability applied to agent semantics (tool calls, LLM requests, human approval), with
isolated execution and full observability.

> Status: early design / documentation. Personal engineering project — v1 scope.

## The idea in one line

A workflow's progress is an append-only event log in Postgres. Workers lease jobs and
drive them forward by appending events. When a worker crashes, its lease expires, another
worker re-leases the job, replays the log, skips already-completed steps, and continues —
so **no work is lost and no side effect is duplicated**.

Everything else in the design falls out of that single mechanism.

## Why

Modern AI systems are increasingly long-running, tool-using workflows rather than single
model invocations. Agent workloads run for minutes to hours, call external tools with real
side effects, and consume paid LLM tokens. When a worker crashes midway, the naive outcome
is a lost workflow and — worse — duplicated side effects on retry.

Making those workflows reliable is not an AI problem; it is a distributed-systems problem,
drawing on event sourcing, scheduling, idempotency, isolation, and cloud infrastructure.
Conductor solves that class of problem so a workflow author can focus on agent logic rather
than execution machinery.

## Core mechanism: crash survival by replay

Live workflow state is a deterministic function of a persisted event log. A crashed run is
**resumed, not restarted**. This one mechanism produces three externally-visible behaviours:

- **Crash recovery** — a fresh worker replays the log and continues.
- **Side-effect-safe retries** — completed steps are checkpointed before the next begins, so retries never duplicate side effects.
- **Pause-for-human-approval** — a pause is simply a "waiting for approval" event that withholds the job from scheduling until an approval event is appended.

See the full sequence in [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Capabilities (five pillars)

| Pillar | What it provides |
|--------|------------------|
| Durable execution | Append-only event log; state via deterministic replay; resume-after-crash |
| Distributed scheduling | Work queue with visibility-timeout leasing, priorities, timeouts, retries |
| Secure execution | Per-step Docker sandbox with CPU/memory limits, wall-clock timeout, cleanup |
| Observability | Structured event timeline: LLM requests, tool calls, checkpoints, token cost |
| Cloud platform | AWS (ECS, RDS, S3, SQS, IAM), Terraform, GitHub Actions CI/CD |

## Architecture at a glance

A stateless **API** for submission and streaming, a durable **event store** of record, and a
pool of interchangeable **workers** that lease jobs and drive workflows forward by replaying
and appending events.

| Component | Responsibility | Technology |
|-----------|----------------|------------|
| API | Job submission, status, log streaming (SSE), approvals | FastAPI |
| Event store | Append-only event log; workflow & step tables | PostgreSQL |
| Queue | Job dispatch with visibility-timeout leasing | Redis (local) / SQS (cloud) |
| Workers | Lease jobs, replay history, execute next step, append events | Python worker pool |
| Sandbox | Isolated per-step execution with limits & timeout | Docker |
| Artifact store | Large outputs and logs | S3 |
| Telemetry | Traces, metrics, dashboards | OpenTelemetry + Prometheus + Grafana |

## Documentation

- [Architecture](docs/ARCHITECTURE.md) — components, the central crash-survival mechanism, and the sequence diagram.
- [Roadmap](docs/ROADMAP.md) — phased delivery plan, deliverables, and what's out of scope for v1.
- [Architecture Decision Records](docs/adr/) — key design decisions and their rejected alternatives.

## Design principles

1. **Deterministic** — state is always reconstructable by replaying persisted events.
2. **Durable** — a worker crash never loses completed work.
3. **Observable** — every significant action leaves an inspectable trace with timing, tokens, and cost.
4. **Isolated** — one workflow can never interfere with another.
5. **Minimal** — prefer one mechanism that produces many behaviours over many special-cased features.
