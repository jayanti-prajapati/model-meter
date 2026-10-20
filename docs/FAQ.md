# Frequently Asked Questions

*Last updated: October 2026*

---

## General

### What is Model Meter?

Model Meter is an open-source tool that gives engineering teams visibility into their AI spending. It tracks every LLM API request across multiple providers, attributes costs to specific projects or features, and displays spend trends on a self-hosted dashboard — all with a two-line SDK integration.

### Is Model Meter free?

Yes. Model Meter is free and open-source under the Apache 2.0 license. You can self-host it with no per-seat or per-event charges. There is no hosted SaaS offering at this time; community edition is the only edition.

### Who is Model Meter for?

Model Meter is designed for engineering teams at B2B SaaS companies and AI-native software teams with 5–50 engineers who:
- Use multiple LLM providers (OpenAI, Anthropic, Google, Mistral, etc.)
- Need to attribute AI costs to specific features or projects
- Want to stop surprise AI bills before they arrive on the invoice

It is not designed for individual users tracking personal ChatGPT usage, or for organizations that need full enterprise governance or SSO from day one.

### Is Model Meter an LLM proxy or a gateway?

No. Model Meter is a **cost tracking and visibility tool**, not a proxy or gateway. It does not sit in the critical path of your AI calls. The SDK instruments existing API calls via OpenTelemetry and sends telemetry to Model Meter asynchronously. Your AI requests go directly to the provider.

---

## Privacy and Data

### Does Model Meter store my prompts or completions?

**No.** Model Meter does not collect prompts or completions by default. The SDK only captures:
- Provider and model name
- Token usage (input, output, cached)
- Latency
- Success or error status
- Operation label (optional, set by you)
- Cost estimate

No prompt text, response text, or personally identifying data is transmitted.

### What data does the SDK send to the Model Meter server?

The SDK sends `UsageEventV1` records containing: schema version, source event ID (for deduplication), timestamp, provider, model, operation label, environment, request status, latency, and token counts. Full schema: [docs/INGESTION.md](INGESTION.md).

### Can I self-host Model Meter on my own infrastructure?

Yes, that is the primary deployment model. Run it on Docker Compose locally, on Railway + Neon for a free cloud deployment, or on AWS ECS Fargate + RDS for production. Your usage data never leaves your infrastructure. See [docs/DEPLOYMENT.md](DEPLOYMENT.md).

### Does Model Meter phone home or send telemetry from self-hosted installations?

No. Model Meter does not collect any telemetry from your self-hosted installation.

---

## Setup and Integration

### How long does it take to integrate?

The Docker quickstart takes under 60 seconds. Full SDK integration in an existing Node.js app typically takes 10–15 minutes (install the package, call `init()`, set up the OTel span processor).

### Which JavaScript runtimes and frameworks does the SDK support?

The SDK (`@model-meter/sdk`) is a Node.js package. It works with:
- Any Node.js application (v18+)
- Express, Fastify, NestJS, or plain Node
- Next.js API routes (Node.js runtime only)
- Serverless: AWS Lambda, Vercel Functions (requires explicit `flush()` before the function exits)

The SDK is not currently compatible with Deno, Bun, or browser environments.

### Does Model Meter work with the Vercel AI SDK?

Yes. Model Meter integrates with the Vercel AI SDK via OpenTelemetry. After calling `init()`, set up the OTel provider with `createModelMeterExporter` and call `registerTelemetry(new OpenTelemetry())`. All subsequent `generateText()`, `streamText()`, `generateObject()`, and `embed()` calls are tracked automatically.

### Does Model Meter work with LangChain?

Not yet natively. LangChain emits its own OpenTelemetry spans but uses a different semantic convention from the Vercel AI SDK's `gen_ai.*` attributes. A LangChain adapter is on the roadmap (Milestone 2). In the meantime, you can use the low-level `track()` method to send events manually.

### Can I track AI usage from Python, Go, or other languages?

The TypeScript SDK is the primary integration. For other languages, you can send events directly to the batch ingestion endpoint (`POST /api/v1/usage-events/batch`) with a JSON payload matching the `UsageEventV1` schema. See [docs/API.md](API.md) and [docs/INGESTION.md](INGESTION.md) for the request format.

---

## Cost Tracking

### How does Model Meter calculate costs?

Model Meter uses a built-in pricing catalogue with effective-dated prices per provider and model. The formula for text-token models is:

```
cost = (input_tokens × input_rate / 1,000,000)
     + (output_tokens × output_rate / 1,000,000)
     + (cached_input_tokens × cached_rate / 1,000,000)
```

Calculations use decimal-precision arithmetic (not floating-point) to avoid rounding errors. See [docs/COST-ENGINE.md](COST-ENGINE.md) for the full formula and provenance model.

### Are the cost estimates accurate?

They are estimates, not exact billing amounts. Provider pricing changes, discounts, credits, batch rates, and surcharges are not always captured by request-level estimates. Model Meter marks estimates clearly as `estimated` vs. `provider_reported` vs. `reconciled`. The billing reconciliation feature (Milestone 5) will let you import provider invoices and compare them to estimates.

### What happens when a model's price changes?

Model Meter uses effective-dated pricing records. Each cost record stores the pricing version that was active at the time of the request, so historical cost data is not silently rewritten when prices change. New pricing records are added with an effective start date.

### What happens if a model is not in the pricing catalogue?

Events for unknown models are still ingested and stored. Cost fields will be null or marked as unpriced rather than assigned a guessed rate. You can contribute pricing records for missing models — see [CONTRIBUTING.md](../CONTRIBUTING.md).

### How does Model Meter handle cached tokens?

Cached input tokens (e.g., Anthropic's prompt caching, OpenAI's cached prompts) are tracked separately in the `cachedInputTokens` field. The cost engine applies the correct cached-token rate and does not double-count cached tokens as regular input tokens.

---

## Architecture and Deployment

### What database does Model Meter use?

PostgreSQL 16. The API runs all database migrations automatically on startup using Knex. No manual migration step is needed for standard deployments.

### Can I run Model Meter without Docker?

Yes. For local development without Docker: start Postgres separately, copy `.env.example` to `.env`, configure `DATABASE_URL`, and run `yarn workspace @model-meter/api dev`. See [AGENTS.md](../AGENTS.md) for full commands.

### Is Model Meter multi-tenant?

Yes. The data model supports multiple workspaces with complete tenant isolation. Every database query is workspace-scoped. PostgreSQL Row-Level Security (RLS) is used as defense-in-depth. See [docs/SECURITY.md](SECURITY.md) for details.

### How does Model Meter handle high event volume?

The SDK buffers events locally and flushes in batches (every 5 seconds or 50 events). The API validates and persists events; the worker calculates costs and updates daily aggregates. The dashboard reads pre-aggregated data for fast queries. For high-volume deployments, the architecture supports horizontal scaling of the API and worker independently.

---

## Comparison

### How does Model Meter compare to Helicone?

Both tools track LLM usage and cost. Key differences:
- **Model Meter** is fully open-source (Apache 2.0), self-hostable with no hosted dependency, and does not act as a proxy. **Helicone** offers a hosted service and uses a proxy model.
- **Model Meter** focuses on cost attribution to projects/features and team-level spend. Helicone covers prompt debugging and caching in addition to observability.
- **Model Meter** does not store prompts or completions by default. Helicone stores request/response data by default.

See [docs/COMPARISON.md](COMPARISON.md) for a full side-by-side comparison.

### How does Model Meter compare to LangSmith?

LangSmith is primarily a **prompt engineering and evaluation platform** (tracing, annotation, datasets, evals). Model Meter is primarily a **cost visibility and spend attribution tool**. They address different problems:
- Use LangSmith if you need to debug prompts, annotate traces, or run automated evaluations.
- Use Model Meter if you need to track spending by project, alert on budget thresholds, or reconcile AI costs with provider invoices.
- The tools are complementary — you can use both.

---

## Contributing

### How do I contribute?

1. Read [CONTRIBUTING.md](../CONTRIBUTING.md).
2. Browse [`good first issue`](https://github.com/jayanti-prajapati/model-meter/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22) on GitHub.
3. Open an issue to discuss large changes before implementing.

Good starting points: adding or verifying provider pricing fixtures, writing tests, or improving documentation.

### How do I add pricing for a missing provider or model?

Pricing fixtures live in `packages/cost-engine/src/`. Each record includes provider, model identifier, effective date, input/output/cached token rates, and a source URL. Add a new record and include a test asserting the calculated cost for a known input. See the existing fixtures for the format, then open a PR.

### Can I request a provider integration or gateway adapter?

Yes — open a [feature request](https://github.com/jayanti-prajapati/model-meter/issues/new?template=feature_request.md) describing the provider and what data it exposes. Gateway adapters (LiteLLM, Portkey, etc.) are on the roadmap for Milestone 2.
