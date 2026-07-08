# ADR 0003 — Orchestration: AWS ECS

**Status:** Accepted

## Context

The API and worker pool need to run in the cloud with reproducible infrastructure and
automated delivery. The choice of orchestration platform should cover the relevant
distributed-systems concepts without imposing operational overhead disproportionate to a
single-author project.

## Decision

Deploy to **AWS ECS**, with state in RDS (PostgreSQL), artifacts in S3, and a managed queue
via SQS — all defined in Terraform and shipped through GitHub Actions.

## Alternatives considered

- **Kubernetes.** Rejected. It adds unnecessary operational surface for a single-author
  project; ECS covers the same concepts with less yak-shaving.

## Consequences

- Reproducible, infrastructure-as-code deployments via Terraform.
- Lower operational burden than a self-managed Kubernetes cluster.
- Some portability trade-off from adopting AWS-managed services (ECS, RDS, SQS).
