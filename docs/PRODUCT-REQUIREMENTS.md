# Product Requirements — Model Meter Community Edition

## 1. Product summary

Model Meter gives small software teams one centralized view of AI usage, spend, latency, errors, and task-level quality across multiple LLM providers and applications.

## 2. Target customer

Initial ideal customer:
- B2B SaaS company or AI-native software team with approximately 5–50 engineers.
- Uses multiple LLM models/providers or has several AI-powered features.
- Calls provider APIs from production applications or routes traffic through a gateway.
- Needs to attribute costs to projects/features and explain unexpected bills.
- Does not have a dedicated AI infrastructure/FinOps team.

Not the first target:
- Consumers tracking personal ChatGPT subscriptions.
- Organizations that require full enterprise governance, SSO, advanced compliance, or complex chargeback from day one.
- Teams expecting all closed AI coding tools to expose request-level telemetry.

## 3. User roles

- **Owner:** Workspace settings, members, integrations, budgets, data lifecycle.
- **Admin:** Manage projects, integrations, budgets, and members as permitted.
- **Engineer:** View usage and traces for permitted projects; manage approved project settings.
- **Finance:** View spend, budgets, imports, and reconciliation reports.
- **Viewer:** Read-only access to permitted workspace data.

Role permissions must be enforced by the API, not just by hiding UI controls.

## 4. Jobs to be done

1. When the AI bill increases, identify which project, model, or operation caused the change.
2. When launching an AI feature, estimate and monitor its cost per request or successful task.
3. When choosing a model, compare cost, latency, and task quality on equivalent evaluation cases.
4. When finance asks for actual spend, compare request-level estimates with provider-reported charges.
5. When usage approaches a budget, alert the responsible team before the limit is exceeded.

## 5. Functional requirements

### Usage collection
- Accept validated, versioned events from SDKs and supported gateway adapters.
- Accept batch ingestion with per-event validation results.
- Identify source, integration, provider, model, project, environment, operation, timestamp, and status.
- Preserve provider-specific billable usage dimensions.
- Support idempotent retries and deduplication.
- Expose ingestion health and last-seen timestamps.

### Cost calculation
- Use effective-dated pricing records.
- Support input/output/cached token prices initially.
- Extend to provider-specific token categories, images, audio, batch pricing, and other billable units.
- Store pricing version and cost provenance.
- Distinguish estimated request cost from provider-billed cost.
- Do not silently rewrite historical costs when pricing changes.

### Analytics
- Workspace summary: spend, request count, errors, latency, budget consumption.
- Spend time series with explicit timezone handling.
- Breakdowns by provider, model, project, environment, and operation.
- Filterable request/event explorer.
- Clear data freshness and missing-data indicators.

### Budgets and alerts
- Workspace and project budgets.
- Daily and monthly periods initially.
- Configurable thresholds and alert delivery status.
- Idempotent alert evaluation and notification delivery.

### Billing reconciliation
- Import provider-reported usage/billing data from supported sources.
- Preserve source identifiers, billing period, currency, and import status.
- Show differences without adding imported invoice totals to request-level estimates.
- Support retries, pagination, late adjustments, and import checkpoints.

### Cost-to-quality
- Run comparable evaluation datasets across candidate models.
- Record task, dataset version, model, quality metric, cost, latency, and evaluation time.
- Recommend lower-cost candidates only when defined quality and latency constraints are met.
- Begin with offline evaluations and human-approved recommendations.

## 6. Non-functional requirements

- Tenant isolation and least-privilege access.
- No prompt/completion collection by default.
- TLS in transit and encryption at rest.
- Idempotent ingestion and recoverable background processing.
- Observable API/worker health and failed jobs.
- Documented backup, restore, retention, and deletion procedures.
- Bounded request sizes and configurable rate limits.
- Stable versioned API and event schemas.

## 7. Explicit non-goals for the first MVP

- Building a universal LLM proxy.
- Automatically switching production models without customer approval.
- Supporting every provider and every billable modality.
- Storing prompts and completions by default.
- Enterprise SSO, complex chargeback, or advanced compliance certifications.
- Replacing full-featured LLM observability platforms.
