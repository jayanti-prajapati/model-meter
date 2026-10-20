# Deployment Plan

## Local development

Proposed prerequisites:
- Node.js LTS and Corepack/Yarn.
- Docker and Docker Compose.
- PostgreSQL.
- AWS account only for testing AWS-specific integrations.

Use `.env.example` as a template. Never commit `.env` or real credentials.

## AWS environments

Keep development, staging, and production isolated. Use separate databases, queues, secrets, and IAM roles.

## Network layout

- Public edge: CloudFront and HTTPS entry point.
- API/worker: ECS Fargate tasks in private subnets.
- Database: RDS PostgreSQL in private subnets, no public access.
- Queue: SQS with a dead-letter queue.
- Imports: private S3 bucket for temporary billing files if needed.
- Secrets: Secrets Manager.
- Encryption: KMS.
- Monitoring: CloudWatch logs, metrics, alarms.

## CI/CD

Recommended checks for each pull request:
1. Formatting and lint.
2. Type checking.
3. Unit tests.
4. Cost-engine fixtures.
5. API integration tests.
6. Tenant isolation/RLS tests.
7. Dependency and secret scanning.
8. Build web, API, and worker images.

Deploy migrations as a controlled step with a rollback/forward-fix plan. Use backward-compatible migrations when API and worker versions may overlap during rollout.

## Operational monitoring

Track:
- API latency and error rate.
- Ingestion accepted/rejected counts.
- Queue depth and oldest message age.
- Worker failures and DLQ count.
- Unknown model/pricing count.
- Last successful billing import.
- Aggregate freshness.
- Database connections, CPU, storage, and slow queries.
- Budget alert delivery failures.

## Recovery

Document RPO/RTO targets before production. Test database restore and replay procedures. Ensure replaying events cannot double-count aggregates or budget alerts.
