# API Design

Base path: `/api/v1`. JSON over HTTPS. Use Zod schemas and stable error shapes. All timestamps are ISO-8601; all date ranges are explicit and timezone-aware.

## Authentication

- Dashboard: OIDC/Cognito-backed session or validated access token.
- Ingestion: scoped write-only ingestion key.
- Provider imports: background worker using a secret reference.
- Internal jobs: AWS IAM task roles.

Resolve the workspace from the authenticated identity/credential. Do not accept a caller-provided workspace ID as authorization.

## Planned endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/usage-events` | Submit one event |
| POST | `/usage-events/batch` | Submit event batch |
| GET | `/dashboard/summary` | KPIs for a date range |
| GET | `/analytics/spend` | Spend time series |
| GET | `/analytics/breakdown` | Breakdown by provider/model/project |
| GET | `/usage-events` | Filterable event explorer |
| GET / POST | `/projects` | List/create projects |
| PATCH | `/projects/:id` | Update project |
| GET / POST | `/integrations` | List/create integrations |
| POST | `/integrations/:id/test` | Test integration |
| GET / POST | `/budgets` | List/create budgets |
| PATCH | `/budgets/:id` | Update budget |
| GET | `/alerts` | List alerts |
| GET / POST | `/members` | List/invite members |
| DELETE | `/members/:userId` | Remove member |
| GET | `/reconciliation/runs` | Import/reconciliation status |

## Ingestion request example

```json
{
  "events": [
    {
      "schemaVersion": 1,
      "sourceEventId": "provider-request-123",
      "occurredAt": "2026-10-14T12:30:00Z",
      "provider": "example-provider",
      "model": "example-model",
      "operation": "answer-ticket",
      "environment": "production",
      "status": "success",
      "latencyMs": 840,
      "usage": {
        "inputTokens": 1200,
        "outputTokens": 350
      }
    }
  ]
}
```

## Response semantics

Return HTTP 202 only after events are durably accepted into the configured ingestion path. Include accepted IDs and per-event rejection reasons. Use 400 for invalid schema, 401 for missing/invalid credentials, 403 for unauthorized scope, 413 for oversized batches, 429 for rate limits, and 5xx for server failures.

Define a maximum batch size and request body size. Support partial batch validation only if clients can reliably identify rejected records and retry safely.

## Query parameters

Analytics endpoints should support:
- `from`, `to` — required or defaulted explicit time bounds.
- `timezone` — validated IANA timezone for grouping.
- `projectId`, `provider`, `model`, `environment`.
- `groupBy` — allowlisted dimensions only.
- `cursor` and `limit` for event lists.

Never interpolate user-provided column names into SQL. Map `groupBy` values to a fixed allowlist.

## API security

- Validate all inputs.
- Rate-limit by workspace and credential.
- Use request IDs and structured audit logs.
- Avoid returning secrets or sensitive provider payloads.
- Ensure all queries are tenant-scoped and covered by RLS.
- Use cursor pagination for large event lists.
