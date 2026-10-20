# Architecture

## Architecture decision

Start with a **modular monolith**: one TypeScript repository with separate web, API, and worker processes. Share contracts and business logic through packages. Deploy API and worker independently from the same codebase.

Avoid microservices, Kubernetes, Kafka, and a separate analytical warehouse until measured scale or reliability requirements justify them.

## Logical components

1. **Web app:** React + Vite + TypeScript + MUI. Uses authenticated APIs; never accesses provider secrets.
2. **API:** Express + TypeScript. Authentication, workspace authorization, validation, management endpoints, analytics reads, ingestion.
3. **Contracts:** Versioned Zod schemas for ingestion events and API payloads.
4. **Ingestion:** Authentication, rate limiting, validation, normalization, source identity, idempotency.
5. **Queue:** SQS in AWS. Local development may use a compatible queue adapter or a simple worker mode.
6. **Worker:** Processes normalized events, resolves pricing, persists events, updates aggregates, evaluates budgets, handles imports.
7. **Database:** PostgreSQL for tenant data, usage events, pricing, aggregates, budgets, billing imports, audit records.
8. **Cost engine:** Pure functions with versioned pricing inputs and deterministic tests.
9. **Integrations:** Gateway callbacks, SDK instrumentation, provider billing imports.
10. **Infrastructure:** ECS Fargate, RDS PostgreSQL, SQS, S3, CloudFront, Secrets Manager, KMS, CloudWatch.

## AWS deployment topology

- CloudFront serves the static frontend from a private S3 origin.
- API and worker run as separate ECS Fargate services in private subnets.
- An Application Load Balancer or API Gateway exposes the API over HTTPS.
- RDS PostgreSQL resides in private subnets and is not publicly accessible.
- SQS queues use dead-letter queues and least-privilege access policies.
- EventBridge Scheduler triggers billing import jobs and reconciliation.
- Secrets Manager stores provider credentials; KMS provides encryption controls.
- CloudWatch collects structured logs, metrics, alarms, and worker failure signals.
- Use VPC endpoints where appropriate to reduce public egress and improve network controls.

Choose ALB versus API Gateway based on traffic patterns, authentication, quotas, and operational cost. The first release needs only one externally exposed API surface.

## Data flow

1. Customer creates a project and scoped ingestion key.
2. SDK/gateway sends event batch over HTTPS.
3. API authenticates the key, resolves integration/project, validates schema, applies rate limits, and assigns server-owned tenant context.
4. Valid events enter a durable handoff path and SQS.
5. Worker deduplicates by integration and source event identity.
6. Cost engine uses the correct effective-dated pricing record and billable dimensions.
7. Worker writes normalized event and updates aggregates idempotently.
8. Budget evaluator records threshold crossings and schedules notifications.
9. Dashboard queries pre-aggregated data and offers event-level drill-down.
10. Billing import jobs run separately and reconcile provider-reported data.

## Reliability and delivery semantics

SQS is at-least-once delivery. Assume messages can be delivered more than once and out of order.

- Database unique constraints protect event identity.
- Worker retries transient failures with backoff.
- Poison messages go to a dead-letter queue after a bounded number of attempts.
- Acknowledge queue messages only after durable processing or handoff.
- If API acceptance requires strong durability across database and queue publishing, use a PostgreSQL transactional outbox.
- Aggregates must be replayable or correctable after late events and pricing corrections.

## Scaling path

1. Start with PostgreSQL indexes and daily aggregates.
2. Add connection pooling, query profiling, and retention controls.
3. Partition usage events by time when volume warrants it.
4. Add a dedicated analytical store only if PostgreSQL no longer meets measured dashboard/reporting needs.
5. Separate high-volume ingestion from dashboard API if load patterns require it.
