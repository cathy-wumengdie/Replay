# Roadmap & Delivery Plan

The hard, differentiating work is sequenced first, while motivation is highest. Cloud and
observability polish come last, where scope can safely be trimmed. Each phase is
independently demonstrable.

> The phase-level plan below is broken down into concrete, 2-8h issues in
> [BACKLOG.md](BACKLOG.md), which mirrors the [GitHub Issues](https://github.com/cathy-wumengdie/Replay/issues)
> and [Milestones](https://github.com/cathy-wumengdie/Replay/milestones) (M0–M5).

## Phases

| Phase | Goal | Demonstrates |
|-------|------|--------------|
| 1 — Engine (local) | Single worker runs a workflow, is killed mid-run, resumes from the event log | Event sourcing, deterministic replay — the hardest 30% |
| 2 — Distribution | Queue + multiple workers; lease expiry re-dispatches crashed jobs | At-least-once delivery, leasing, idempotency |
| 3 — Isolation | Steps execute in Docker with limits, timeout, cleanup | Sandboxing, resource control |
| 4 — API + HITL | FastAPI submission, SSE streaming, pause-for-approval flow | Public API, human-in-the-loop, streaming |
| 5 — Cloud + Observability | Terraform to AWS, GitHub Actions CI/CD, OTel/Prometheus/Grafana | IaC, CI/CD, production telemetry |

**Next step:** finalise the Phase 1 data model — the event-log schema and the workflow/step
tables — from which Phases 2–4 follow directly.

## Deliverables

- Working platform deployed to AWS, with a demo workflow that survives an induced crash.
- Architecture document (expanded) with a crash-survival sequence diagram as its centrepiece.
- ADR log recording the design decisions and their rejected alternatives.
- README + demo with a one-line description and a short recorded walkthrough of a
  crash-and-resume.

## Out of scope (v1)

Explicitly deferred to keep v1 finishable and focused on the differentiating core:

- Autoscaling workers on queue depth (design for it; do not build it yet).
- Multi-tenant admin UI and full user-management.
- Self-hosted / BYOC distribution — v1 is a single hosted deployment.
- Kubernetes and Kafka (see [ADR 0002](adr/0002-queue-technology.md) and [ADR 0003](adr/0003-orchestration.md)).

## Users

| Role | Cares about | Interacts via |
|------|-------------|---------------|
| AI developer (primary) | Defining workflows, submitting jobs, streaming progress, debugging, cost | SDK + public API + timeline UI |
| Platform engineer | Scaling, worker health, deployment, isolation | Infrastructure (Terraform, dashboards) |
| Administrator | Quotas, budget limits, kill switches, failure debugging | Admin controls + observability stack |

## Engineering concepts demonstrated

| Concept | Where it appears |
|---------|------------------|
| Event sourcing | Append-only log as store of record; state via replay |
| Deterministic replay | Reconstructing live state from history |
| Durable execution | Resume-after-crash instead of restart |
| Distributed scheduling | Queue, priorities, timeouts |
| Leasing / visibility timeout | Crashed-worker jobs re-dispatched safely |
| Idempotency | Checkpoint-aware retries; no duplicated side effects |
| Failure recovery | The central crash-survival flow |
| Human-in-the-loop | Pause-for-approval as a workflow event |
| Docker isolation | Per-step sandboxing with limits and timeouts |
| Cloud deployment (AWS) | ECS, RDS, S3, SQS, IAM |
| Infrastructure as code | Terraform, reproducible environments |
| CI/CD | GitHub Actions delivery pipeline |
| Observability | OpenTelemetry, Prometheus, Grafana, cost attribution |
