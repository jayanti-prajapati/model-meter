# Changelog

All notable changes to Model Meter are documented here.

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
Versioning: [Semantic Versioning](https://semver.org/spec/v2.0.0.html)

---

## [Unreleased]

### Planned
- Auth and workspace UI (M1)
- Gateway adapters: LiteLLM, Portkey (M2)
- Workspace and project budget alerts (M3)
- Provider billing import and reconciliation (M4)
- Cost-to-quality model evaluation (M5)

---

## [0.1.0] — 2026-10-16

### Added

#### Monorepo scaffold
- Yarn 4 Berry + Turborepo monorepo with shared TypeScript configuration
- `turbo.json` pipeline: `build → test → check-types`
- GitHub Actions CI workflow: type-check → test → build

#### `@model-meter/contracts`
- Versioned Zod schema for `UsageEventV1`: provider, model, operation, environment, token usage, latency, status, idempotency
- Ingestion batch schema with per-event validation
- Analytics query schemas

#### `@model-meter/database`
- Knex client with auto-migration on API startup
- 5 migrations covering: workspaces, users, workspace members, projects, integrations, ingestion keys, usage events, model prices, daily aggregates, budgets, alerts
- `upsertUsageEvent` and `upsertDailyAggregate` repositories with idempotency

#### `@model-meter/cost-engine`
- `calculateCost()` — decimal-precision cost calculation using `decimal.js`
- Effective-dated pricing support
- Deterministic, independently tested pure functions

#### `@model-meter/sdk` (npm: `@model-meter/sdk`)
- `init()` — one-call setup with API key, endpoint, project ID, environment
- Auto-flush event buffer (every 5 seconds or 50 events)
- OpenTelemetry span exporter — captures `gen_ai.*` spans automatically from the Vercel AI SDK
- `generateText()` and `streamText()` wrappers for operation-level labeling
- Graceful failure: SDK errors never surface to the host application

#### `apps/api`
- Express server with auto-migration on startup
- `POST /api/v1/usage-events/batch` — authenticated batch ingestion with per-event validation
- Analytics endpoints: `/dashboard/summary`, `/analytics/spend`, `/analytics/breakdown`, `/usage-events`
- Projects API: list, create, update
- `GET /health` — liveness check with database connectivity status

#### `apps/worker`
- Background event processor (in-process; SQS-ready interface for future scale)
- Cost engine integration, aggregate upsert, deduplication

#### `apps/admin`
- React 18 + Vite + shadcn/ui dashboard
- Spend overview, time-series chart, provider/model/project breakdown, event explorer
- Projects management page

#### `apps/example`
- Demo Express app using `@model-meter/sdk` to instrument AI calls
- Browser UI at port 3001

#### Infrastructure
- `docker/docker-compose.yml` — full stack: Postgres + API + admin
- Dockerfiles for API and admin
- `.env.example` documenting all environment variables

### Known gaps in v0.1.0
- No workspace signup or login UI — ingestion keys require manual DB seeding
- No key management UI in the admin
- Worker process is currently run inline in the API (separate process interface preserved for M2)
- Admin shows empty state until an ingestion key is seeded manually

---

[Unreleased]: https://github.com/jayanti-prajapati/model-meter/compare/v0.1.0...HEAD
[0.1.0]: https://github.com/jayanti-prajapati/model-meter/releases/tag/v0.1.0
