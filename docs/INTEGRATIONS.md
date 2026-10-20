# Integrations and Provider Compatibility

Model Meter tracks AI spending across providers and collection methods. This document lists supported providers, gateway adapters, and planned integrations.

*Last updated: October 2026. Provider feature sets change frequently — check provider documentation for the latest billing specifics.*

---

## Collection Methods

Model Meter supports three paths for collecting usage data:

| Method | How it works | Best for |
|---|---|---|
| **TypeScript SDK** | Instruments your Node.js app via OpenTelemetry | Teams using the Vercel AI SDK or calling provider APIs directly |
| **Gateway adapter** | Receives telemetry emitted by an LLM gateway | Teams routing traffic through LiteLLM, Portkey, or similar |
| **Billing import** | Imports provider-issued invoice/usage data | Finance reconciliation and actual-vs-estimated comparisons |

---

## TypeScript SDK: Provider Compatibility

The SDK captures usage from **any provider supported by the Vercel AI SDK** via OpenTelemetry `gen_ai.*` spans. You do not need a Model Meter provider integration for each provider — the SDK reads whatever the Vercel AI SDK reports.

### Verified providers

| Provider | Package | Tokens captured | Cost estimate |
|---|---|---|---|
| OpenAI | `@ai-sdk/openai` | Input, output, cached | Yes |
| Anthropic | `@ai-sdk/anthropic` | Input, output, cached | Yes |
| Google Gemini | `@ai-sdk/google` | Input, output | Yes |
| Mistral | `@ai-sdk/mistral` | Input, output | Yes |
| Cohere | `@ai-sdk/cohere` | Input, output | Yes |
| Azure OpenAI | `@ai-sdk/azure` | Input, output, cached | Yes (uses OpenAI pricing) |
| Amazon Bedrock | `@ai-sdk/amazon-bedrock` | Input, output | Partial — pricing catalogue incomplete |
| Groq | `@ai-sdk/groq` | Input, output | Yes |
| Together AI | (via OpenAI compat.) | Input, output | Partial |

**Notes:**
- "Cached" tokens are only captured when the provider and SDK version expose `cachedInputTokens` in the span.
- "Partial" pricing means the model is tracked but cost may be zero if the price record is missing. Contributions welcome.
- Providers not listed may still work if they expose `gen_ai.*` OTel attributes — open an issue to request verification.

### Manual tracking (any language)

For providers or frameworks not covered by the Vercel AI SDK, use the HTTP ingestion API directly:

```bash
curl -X POST http://localhost:3000/api/v1/usage-events/batch \
  -H "Authorization: Bearer <your-ingestion-key>" \
  -H "Content-Type: application/json" \
  -d '{
    "events": [{
      "schemaVersion": 1,
      "sourceEventId": "my-unique-request-id",
      "occurredAt": "2026-10-20T12:00:00Z",
      "provider": "openai",
      "model": "gpt-4o",
      "operation": "my-feature",
      "environment": "production",
      "status": "success",
      "latencyMs": 1200,
      "usage": {
        "inputTokens": 800,
        "outputTokens": 250
      }
    }]
  }'
```

See [docs/API.md](API.md) for the full schema and [docs/INGESTION.md](INGESTION.md) for field semantics.

---

## Gateway Adapters

Gateway adapters normalize telemetry from LLM gateways into Model Meter's canonical event format.

| Gateway | Status | Notes |
|---|---|---|
| [LiteLLM](https://litellm.ai) | 🔜 Milestone 2 | Callback-based; exposes per-request cost and token data |
| [Portkey](https://portkey.ai) | 🔜 Milestone 2 | Webhook-based; provides detailed usage metadata |
| [OpenRouter](https://openrouter.ai) | 🔜 Planned | REST-based usage data; provider normalization needed |
| [Braintrust Proxy](https://www.braintrust.dev) | 🔜 Planned | |
| [Helicone](https://helicone.ai) | 🔜 Planned | Proxy-mode data export |

**Using a gateway not listed?** Open a [feature request](https://github.com/jayanti-prajapati/model-meter/issues/new?template=feature_request.md) with details about what telemetry data the gateway exposes (webhook format, fields available, auth mechanism).

---

## Billing Imports

Billing imports pull provider-reported usage and invoice data for reconciliation against Model Meter's request-level estimates.

| Provider | Status | Notes |
|---|---|---|
| OpenAI Usage API | 🔜 Milestone 5 | Paginated usage records, cost by model |
| Anthropic Usage API | 🔜 Planned | |
| Google Cloud Billing | 🔜 Planned | BigQuery export or billing API |
| AWS Bedrock Cost Explorer | 🔜 Planned | |
| Azure Cost Management | 🔜 Planned | |

Billing import records are stored separately from request-level estimates. They are never summed together. The reconciliation report shows both sources side-by-side and flags differences.

---

## Pricing Catalogue Coverage

Cost estimates depend on the provider+model having a pricing record in the catalogue. Current coverage as of v0.1.0:

| Provider | Models with pricing |
|---|---|
| OpenAI | gpt-4o, gpt-4o-mini, gpt-4-turbo, gpt-3.5-turbo, text-embedding-3-small, text-embedding-3-large |
| Anthropic | claude-3-5-sonnet, claude-3-5-haiku, claude-3-opus, claude-3-sonnet, claude-3-haiku |
| Google | gemini-1.5-pro, gemini-1.5-flash, gemini-2.0-flash |
| Mistral | mistral-large-latest, mistral-small-latest, mistral-8x7b |
| Groq | llama3-70b-8192, mixtral-8x7b-32768 |
| Cohere | command-r-plus, command-r |

Events for models not in the catalogue are stored with `estimated_cost_usd = null`. You can contribute pricing records — see [CONTRIBUTING.md](../CONTRIBUTING.md).

---

## SDK Framework Compatibility

| Framework / Runtime | Support | Notes |
|---|---|---|
| Node.js 18+ | Full | All features |
| Node.js 22 (recommended) | Full | All features |
| Next.js (Node.js runtime) | Full | Call `init()` in `instrumentation.ts` |
| Next.js (Edge runtime) | Not supported | Edge runtime lacks Node.js APIs |
| AWS Lambda | Full (with `flush()`) | Call `await flush()` before handler returns |
| Vercel Functions | Full (with `flush()`) | Call `await flush()` before function returns |
| Cloudflare Workers | Not supported | Uses V8 isolates, not Node.js |
| Deno | Not supported | Planned |
| Bun | Not tested | May work; open an issue |
| Browser | Not supported | Use the HTTP API from a backend proxy |
