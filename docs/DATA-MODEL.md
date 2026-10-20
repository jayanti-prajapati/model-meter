# Data Model

This document describes the proposed PostgreSQL schema. It is a design plan, not a claim that migrations have already been implemented.

## Main entities

- `workspaces`: customer tenants.
- `users`: local identity mapping or user profile.
- `workspace_members`: user membership and role.
- `projects`: application, feature, or environment grouping.
- `integrations`: SDK, gateway, or billing connection.
- `ingestion_keys`: hashed, scoped, revocable write credentials.
- `usage_events`: normalized request-level events.
- `usage_dimensions`: optional provider-specific billable unit details.
- `model_prices`: effective-dated price catalogue.
- `usage_daily_aggregates`: dashboard-ready rollups.
- `budgets`: workspace/project limits.
- `alerts`: threshold crossings and delivery status.
- `billing_import_runs`: checkpoints and status for import jobs.
- `billing_records`: provider-reported billable records.
- `reconciliation_results`: comparison between estimates and billing.
- `evaluations`: model quality evaluation runs.
- `evaluation_results`: per-case model output metrics (avoid storing sensitive output by default).
- `audit_events`: security-relevant changes.

## Core constraints

- Use UUID primary keys.
- Tenant-owned rows carry `workspace_id`.
- Use composite foreign keys to ensure a project belongs to the same workspace as its child record.
- Add uniqueness for `(integration_id, source_event_id)`.
- Use `TIMESTAMPTZ` for event times.
- Use `NUMERIC` or integer minor units for monetary values, never floating-point money.
- Keep source/provenance and cost calculation version.
- Preserve raw provider records only where necessary, encrypted, access-controlled, and with retention limits.

## Proposed usage table

```sql
CREATE TABLE usage_events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  workspace_id UUID NOT NULL REFERENCES workspaces(id),
  project_id UUID NOT NULL,
  integration_id UUID NOT NULL REFERENCES integrations(id),
  source TEXT NOT NULL,
  source_event_id TEXT NOT NULL,
  provider TEXT NOT NULL,
  model TEXT NOT NULL,
  operation TEXT,
  environment TEXT,
  input_tokens BIGINT,
  output_tokens BIGINT,
  cached_input_tokens BIGINT,
  reasoning_tokens BIGINT,
  latency_ms INTEGER,
  status TEXT NOT NULL CHECK (status IN ('success', 'error')),
  estimated_cost_usd NUMERIC(20, 10),
  pricing_version_id UUID,
  occurred_at TIMESTAMPTZ NOT NULL,
  ingested_at TIMESTAMPTZ NOT NULL DEFAULT now(),
  UNIQUE (integration_id, source_event_id),
  FOREIGN KEY (workspace_id, project_id)
    REFERENCES projects(workspace_id, id)
);

CREATE INDEX idx_usage_workspace_time
  ON usage_events(workspace_id, occurred_at DESC);

CREATE INDEX idx_usage_project_time
  ON usage_events(workspace_id, project_id, occurred_at DESC);
```

## Cost provenance

A cost record should distinguish:
- `estimated`: calculated from request usage and Model Meter pricing.
- `provider_reported`: supplied by provider usage/billing API.
- `reconciled`: matched or adjusted through a documented reconciliation process.

Don't overwrite estimates with provider invoices. Keep the two sources visible and explain the difference.

## Aggregations

Daily aggregates should include dimensions such as workspace, project, date, provider, model, and environment, along with request counts, error counts, token totals, cost totals, and suitable latency statistics. Updates must be idempotent and support late-arriving events.

## Retention and partitioning

Begin with indexed PostgreSQL tables and documented retention. Add time partitioning when event volume justifies it. Avoid premature partitioning before migrations, deletes, foreign keys, and query patterns are tested.
