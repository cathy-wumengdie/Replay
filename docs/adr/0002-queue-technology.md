# ADR 0002 — Queue technology: Redis → SQS

**Status:** Accepted

## Context

Workers acquire jobs from a queue using time-limited leases. A crashed or hung worker's
lease must expire so the job becomes visible again for another worker to pick up. This
requires per-message acking and visibility-timeout semantics. The platform needs a
lightweight option for local development and a managed option for cloud deployment.

## Decision

Use **Redis for local development** and **Amazon SQS in the cloud**. Both provide the
per-message acking and visibility-timeout (leasing) semantics that job dispatch depends on.

## Alternatives considered

- **Kafka.** Rejected. Kafka is a high-throughput log, not a work queue; it lacks the
  per-message acking and visibility semantics needed for job leasing.

## Consequences

- Visibility-timeout leasing is available in both environments, keeping dev and cloud
  behaviour aligned.
- Managed SQS removes queue operations from the cloud deployment.
- The worker's queue interface must abstract over the two backends.
