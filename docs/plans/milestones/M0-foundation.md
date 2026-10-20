# M0 — Foundation

**Status:** ✅ Shipped  
**Branch:** `main`  
**Completed:** October 2026

---

## Scope

Everything needed to run the platform locally end-to-end and demonstrate the core tracking loop.

---

## What was built

### Monorepo scaffold

- Yarn 4 Berry + Turborepo
- TypeScript throughout (`module: NodeNext`, ESM)
- Shared `packages/typescript-config` presets
- `turbo.json` pipeline: `build → test → check-types`

### Packages

| Package | Description |
|---|---|
| `@model-meter/contracts` | Zod schemas for `UsageEventV1`, ingestion batch, analytics queries |
| `@model-meter/database` | Knex client, 5 migrations, `upsertUsageEvent`, `upsertDailyAggregate` |
| `@model-meter/cost-engine` | `calculateCost()` with decimal-precision pricing |
| `@model-meter/sdk` | `init()`, `track()`, auto-flush buffer, OTel span exporter, Vercel AI wrappers |

### Apps

| App | Description |
|---|---|
| `apps/api` | Express server — auto-migrates Postgres on startup, ingestion + analytics + projects routes |
| `apps/worker` | Background event processor (in-process queue; SQS-ready interface) |
| `apps/admin` | React 18 + Vite + shadcn/ui dashboard — spend, breakdown, events, projects |
| `apps/example` | Demo Express app using `@model-meter/sdk`; browser UI at port 3001 |

### Infrastructure

- `docker/docker-compose.yml` — Postgres + API + worker + admin
- Dockerfiles for all four apps
- GitHub Actions CI workflow

---

## Database schema (5 migrations)

| Migration | Tables |
|---|---|
| 001 | `workspaces`, `users`, `workspace_members`, `projects`, `integrations`, `ingestion_keys` |
| 002 | `usage_events`, `model_prices` |
| 003 | `usage_daily_aggregates` |
| 004 | `budgets`, `alerts` |
| 005 | Expression unique index on `usage_daily_aggregates` for `COALESCE(environment, '')` |

---

## Known gaps going into M1

- No workspace signup/login UI — ingestion keys require manual DB seeding
- No key management UI
- Worker process is currently bypassed (events persist inline in the API)
- Admin shows empty state until an ingestion key is seeded
- No user authentication in the admin

---

## Exit criteria (met)

- `yarn build` passes all packages
- `yarn test` — 14/14 tests pass
- API health returns `{"status":"ok","db":"ok"}`
- Events posted via the SDK appear in the admin usage-events table
- Admin dashboard renders without errors
