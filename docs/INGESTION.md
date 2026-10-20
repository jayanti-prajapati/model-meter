# Ingestion and Integrations

## Strategy

Use three collection paths that feed one canonical event contract:

- Gateway events: request-level telemetry when a supported gateway emits it.
- TypeScript SDK: application context and provider response usage.
- Billing imports: provider-reported usage/charges for reconciliation.

These sources are not interchangeable. Keep `source`, source identifiers, provenance, and confidence fields.

## Canonical event

Conceptual TypeScript shape:

```ts
type UsageEventV1 = {
  schemaVersion: 1;
  sourceEventId: string;
  occurredAt: string; // ISO-8601 timestamp
  provider: string;
  model: string;
  operation?: string;
  environment?: string;
  requestCorrelationId?: string;
  status: "success" | "error";
  latencyMs?: number;
  usage?: {
    inputTokens?: number;
    outputTokens?: number;
    cachedInputTokens?: number;
    reasoningTokens?: number;
    // Preserve provider-specific billable dimensions in an extensible map.
    providerSpecific?: Record<string, string | number>;
  };
  // Do not include prompts/completions by default.
};
```

Workspace, project, and integration IDs must be derived from the authenticated ingestion credential or validated server-side mapping. Do not trust tenant IDs sent by an SDK.

## Gateway path

1. Configure gateway callback/export or supported adapter.
2. Create a gateway integration in Model Meter.
3. Authenticate callback using a scoped secret or signed request where supported.
4. Validate payload, timestamp, provider/model identifiers, and source identity.
5. Normalize only fields actually present.
6. Deduplicate and enqueue for processing.

Gateway capabilities differ. Some gateways expose exact usage and cost; others may omit fields or aggregate records. Mark missing fields rather than inventing values.

## TypeScript SDK path

SDK responsibilities:
- Attach project, environment, operation, and correlation metadata.
- Capture latency and success/error status.
- Read provider-reported usage where available.
- Send telemetry asynchronously or through a bounded local buffer.
- Apply timeout/retry/backoff limits.
- Avoid making the customer's main AI operation fail if telemetry is unavailable.
- Never log API keys, prompts, or completions by default.

Provide explicit flush and shutdown hooks for serverless and long-running environments. Document that process termination can lose buffered telemetry unless it has been durably handed off.

## Billing import path

- Use provider OAuth or API credentials with the minimum required scopes.
- Store credentials in a secret store, not in source code or ordinary logs.
- Use paginated, checkpointed imports.
- Preserve provider record IDs, currency, billing period, adjustments, credits, and source timestamps.
- Make imports idempotent.
- Retry rate limits and transient errors with backoff.
- Reconcile separately from request-level estimates; never sum invoice totals into estimated request totals.

## Duplicate prevention

A gateway and SDK may both observe the same underlying request. Prefer an explicit correlation ID propagated across both paths. If there is no reliable correlation, keep both records marked as potentially overlapping rather than silently deduplicating on a risky heuristic. The product must document the selected source-of-truth policy per integration.

## Data quality

Expose:
- Last event received by source.
- Rejected event count and reasons.
- Events with missing usage or unknown model pricing.
- Estimated versus provider-reported cost.
- Import run status and last successful billing sync.
