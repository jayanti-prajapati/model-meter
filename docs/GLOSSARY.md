# Glossary

Definitions for terms used across Model Meter documentation, code, and APIs.

*Last updated: October 2026*

---

## A

### Aggregate (Daily Aggregate)

A pre-computed summary of usage data for a given day, grouped by workspace, project, provider, model, and environment. Aggregates power the dashboard's spend charts and summary KPIs without requiring full table scans. Updated atomically by the worker whenever new events arrive.

### API Key (Ingestion Key)

A write-only credential scoped to a single integration within a workspace. Only a SHA-256 hash is stored in the database — the plaintext is only known at creation time. Used in the `Authorization: Bearer <key>` header when submitting events. Not the same as a provider API key (e.g., your OpenAI key).

---

## B

### Batch Ingestion

Sending multiple usage events in a single HTTP request to `POST /api/v1/usage-events/batch`. Events are validated individually; the response includes accepted IDs and per-event rejection reasons. Preferred over single-event submission for throughput.

### Budget

A workspace- or project-level spending limit, defined as a maximum cost in USD over a daily or monthly period. When spending crosses a configured threshold, an alert is recorded and (in future milestones) a notification is delivered.

---

## C

### Cached Input Tokens

Input tokens served from a provider's prompt cache (e.g., Anthropic Prompt Caching, OpenAI Cached Prompts). Billed at a lower rate than regular input tokens. Tracked separately in the `cachedInputTokens` field and never double-counted as regular input.

### Cost Engine

The `@model-meter/cost-engine` package — a set of pure TypeScript functions that calculate estimated request cost given token counts and an effective-dated pricing record. Uses `decimal.js` for monetary precision. Fully deterministic and independently tested.

### Cost Provenance

The source and confidence level of a cost figure. Model Meter distinguishes three values:
- **`estimated`** — calculated from token counts using Model Meter's pricing catalogue
- **`provider_reported`** — supplied directly by the provider's usage/billing API
- **`reconciled`** — matched or adjusted through a documented reconciliation process

These are never silently mixed. Both estimated and provider-reported costs remain visible and explainable.

---

## D

### Daily Aggregate

See **Aggregate**.

### Dead-Letter Queue (DLQ)

A secondary queue that receives events which the worker has failed to process after a bounded number of retry attempts. Allows failed events to be inspected and replayed without data loss. Relevant for production deployments using SQS.

### Deduplication

The process of preventing the same underlying LLM request from being counted more than once. Model Meter uses the `(integration_id, source_event_id)` composite unique constraint. Retries are safe to submit — `ON CONFLICT DO NOTHING` ensures idempotency.

---

## E

### Effective-Dated Pricing

A pricing record that applies from a specific start date and optionally until an end date. When a model's price changes, a new record is added rather than overwriting the old one. This means historical cost calculations use the price that was active at the time of the request, not the current price.

### Environment

An optional label attached to events identifying the deployment context (e.g., `production`, `staging`, `development`). Used as a filter and breakdown dimension in the dashboard.

### Event Explorer

The dashboard view showing individual usage events with filtering by provider, model, project, environment, and date range. Backed by the `GET /api/v1/usage-events` endpoint with cursor pagination.

---

## G

### Gateway Adapter

An integration module that consumes telemetry emitted by an LLM gateway (e.g., LiteLLM, Portkey) and normalizes it into Model Meter's canonical event format. Planned for Milestone 2. Gateways may expose different fields and aggregation levels than the SDK path.

### `gen_ai.*` Spans

The OpenTelemetry semantic convention for generative AI operations. The Vercel AI SDK emits spans with attributes like `gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`, `gen_ai.request.model`, and `gen_ai.system`. The Model Meter OTel exporter reads these attributes and converts them to usage events.

---

## I

### Idempotency

The property that submitting the same event multiple times produces the same result as submitting it once. Achieved through a unique constraint on `(integration_id, source_event_id)`. Safe retry without risk of double-counting cost or usage.

### Ingestion

The process of receiving, validating, authenticating, and persisting usage events sent by SDKs or gateways. Handled by `apps/api`'s ingestion route and processed by the worker.

### Ingestion Key

See **API Key**.

### Integration

A configured connection between a Model Meter workspace and a data source: an SDK instance, a gateway callback, or a billing import. Each integration has a unique ID that is embedded in every event to identify the source. An ingestion key is scoped to one integration.

---

## L

### Latency (`latency_ms`)

The end-to-end time in milliseconds from when the AI SDK call was made until the response was received. Captured automatically by the OTel exporter. Displayed as median and p95 in the dashboard.

---

## M

### Model Price

A record in the `model_prices` table defining the billable rates for a specific provider and model, effective from a given date. Includes input token rate, output token rate, cached input rate, and billable unit (per token, per 1K tokens, per 1M tokens). Source URL and last verification date are stored for auditability.

### Monorepo

The Model Meter repository structure: a single Git repository containing multiple packages (`contracts`, `database`, `cost-engine`, `sdk-node`) and applications (`api`, `worker`, `admin`, `example`), managed with Yarn 4 Berry workspaces and Turborepo.

---

## O

### Operation

An optional user-defined label for the AI task being performed (e.g., `summarise-ticket`, `generate-email`, `classify-intent`). Set via the `modelMeter.operation` option in the SDK wrappers or the `operation` field in raw events. Enables cost breakdown by feature or task type.

### OpenTelemetry (OTel)

The [OpenTelemetry](https://opentelemetry.io/) observability framework. Model Meter uses OTel's span API to automatically capture `gen_ai.*` attributes emitted by the Vercel AI SDK, eliminating the need to manually instrument each AI call.

---

## P

### Pricing Catalogue

The collection of effective-dated `model_prices` records maintained in the database. Used by the cost engine to look up the correct price for a given provider, model, and event timestamp.

### Project

A logical grouping of AI usage within a workspace, typically corresponding to a product feature, microservice, or team. Projects are the primary attribution dimension for cost tracking.

### Provider

The LLM API provider (e.g., `openai`, `anthropic`, `google`, `mistral`, `cohere`). Stored as a normalized string in usage events. Used as a filter and breakdown dimension in the dashboard.

---

## R

### Reconciled Cost

A cost figure that has been matched or adjusted through the billing reconciliation process (comparing Model Meter's request-level estimates to a provider-issued invoice). See **Cost Provenance**.

### Reconciliation

The process of comparing Model Meter's request-level cost estimates with provider-reported billing data (imported from provider APIs). The reconciliation report shows differences, explains them, and marks records accordingly. Planned for Milestone 5.

### Row-Level Security (RLS)

A PostgreSQL feature that enforces access control at the database row level. Used as defense-in-depth in Model Meter: even if a query bug omits a `workspace_id` filter, RLS prevents cross-tenant data exposure. Applied to all tenant-owned tables.

---

## S

### Schema Version (`schemaVersion`)

An integer field in every usage event indicating the version of the event schema used. Currently `1`. Allows the API to apply version-specific validation and the worker to apply version-specific processing logic as the schema evolves.

### SDK Buffer

The in-memory queue in `@model-meter/sdk` that accumulates events before flushing them to the Model Meter API. Flushed automatically every 5 seconds or when 50 events accumulate. Events in the buffer are lost if the process exits without calling `flush()`.

### Source Event ID (`sourceEventId`)

A caller-provided string uniquely identifying a specific LLM request in the source system. Used for deduplication. Typically the provider's request ID or a UUID generated by the SDK. Required field in `UsageEventV1`.

### Span (OTel Span)

A unit of work in distributed tracing. When using the Vercel AI SDK with the OTel integration, each AI call produces a span containing `gen_ai.*` attributes. The Model Meter OTel exporter reads these spans and converts them to usage events on span completion.

---

## T

### Tenant

A workspace and all its data. Multi-tenant isolation means that workspace A can never read, write, or delete data belonging to workspace B, enforced at both the application layer and the database layer (RLS).

### Token

The basic unit of LLM billing. One token is approximately 0.75 words in English for most models. Providers charge separately for input tokens (your prompt), output tokens (the model's response), and in some cases cached input tokens and reasoning tokens.

---

## U

### Usage Event (`UsageEventV1`)

The canonical data record representing a single LLM API call, as defined in `@model-meter/contracts`. Fields include: schema version, source event ID, timestamp, provider, model, operation, environment, status, latency, token counts, and estimated cost. The source of truth for all cost calculations and dashboard analytics.

---

## W

### Workspace

The top-level organizational unit in Model Meter. Corresponds to a customer account or team. Every resource (projects, integrations, ingestion keys, events, budgets) belongs to exactly one workspace. Workspace IDs are validated server-side — callers cannot claim a workspace they are not authorized for.

### Worker

The `apps/worker` process that consumes usage events, resolves effective-dated pricing, calculates costs with the cost engine, upserts events with idempotency, and updates daily aggregates. Designed to scale independently from the API.
