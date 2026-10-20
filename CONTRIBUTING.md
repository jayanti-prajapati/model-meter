# Contributing to Model Meter

Thanks for considering a contribution. Model Meter is open-source under Apache 2.0 and welcomes contributions of all sizes — from fixing a typo to adding a new gateway adapter.

---

## Before you start

- **Search existing issues** before opening a new one — it may already be tracked.
- **For substantial changes** (new features, schema changes, API changes), open an issue or start a discussion first so the design can be agreed before you invest time implementing it.
- **Never include** real provider API keys, customer data, prompt/completion text, or private billing exports in issues or pull requests.
- **Read the relevant docs** — especially [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) and [docs/DATA-MODEL.md](docs/DATA-MODEL.md) for anything touching the backend.

---

## Development setup

### Prerequisites

- **Node.js 22 LTS** — use `nvm` or similar
- **Corepack / Yarn 4** — `corepack enable` then `yarn --version` should show `4.x`
- **Docker and Docker Compose** — for the local Postgres instance

### Get running

```bash
# 1. Fork and clone the repo
git clone https://github.com/<your-username>/model-meter.git
cd model-meter

# 2. Install all dependencies
yarn install

# 3. Start Postgres
docker compose -f docker/docker-compose.yml up postgres -d

# 4. Configure environment
cp .env.example .env
# Edit .env:
#   DATABASE_URL=postgresql://postgres:postgres@localhost:5432/model_meter
#   INGESTION_KEY_SECRET=<any-random-32-char-string>

# 5. Start the API (auto-migrates on startup)
yarn workspace @model-meter/api dev

# 6. Start the admin UI (separate terminal)
yarn workspace @model-meter/admin dev
```

Admin: http://localhost:5173 | API: http://localhost:3000/health

---

## Common commands

| Command                                        | What it does                       |
| ---------------------------------------------- | ---------------------------------- |
| `yarn build`                                   | Build all packages via Turborepo   |
| `yarn check-types`                             | TypeScript type-check all packages |
| `yarn test`                                    | Run all tests with Vitest          |
| `yarn workspace @model-meter/cost-engine test` | Test a single package              |
| `yarn workspace @model-meter/api dev`          | Start API in watch mode            |
| `yarn workspace @model-meter/admin dev`        | Start admin UI in watch mode       |
| `yarn workspace @model-meter/api db:migrate`   | Run DB migrations manually         |

---

## Repository structure

```
model-meter/
├── apps/
│   ├── api/          # Express ingestion + analytics API
│   ├── admin/        # React / Vite / shadcn/ui dashboard
│   ├── worker/       # Background event processor
│   └── example/      # Demo app using @model-meter/sdk
├── packages/
│   ├── sdk-node/     # @model-meter/sdk — npm package
│   ├── contracts/    # Zod schemas and TypeScript types
│   ├── database/     # Knex client, migrations, repositories
│   ├── cost-engine/  # Pure cost calculation (decimal.js)
│   └── typescript-config/  # Shared tsconfig presets
├── docker/           # docker-compose.yml
├── docs/             # Architecture, API, data model, security
└── .github/          # Issue templates, PR template
```

---

## Development principles

- **TypeScript everywhere** — no plain JavaScript in application code.
- **Validate external inputs** with Zod schemas from `@model-meter/contracts`.
- **Keep cost calculations deterministic** — pure functions with versioned pricing inputs and fixture tests.
- **Preserve tenant boundaries** in every repository method and API query — every query must be workspace-scoped.
- **Make ingestion idempotent** — retries must not duplicate usage or cost records.
- **Avoid prompt/completion capture by default** — do not add fields that would transmit request or response text.
- **Mark planned functionality** as planned — do not imply integrations are complete when they are not.

---

## Branch naming

```
feat/<short-description>     # New feature
fix/<short-description>      # Bug fix
docs/<short-description>     # Documentation only
refactor/<short-description> # Refactor, no behavior change
chore/<short-description>    # CI, deps, tooling
```

Keep branch names lowercase and hyphen-separated.

---

## Commit hygiene

- Write commit messages in the imperative mood: `add cost engine tests`, not `added cost engine tests`.
- Reference the relevant issue: `fix ingestion auth — closes #42`.
- Keep commits focused and atomic. One logical change per commit.
- Do not commit `.env`, secrets, build outputs (`dist/`), or generated billing files.

---

## Pull request checklist

Before marking a PR ready for review, verify:

- [ ] Problem and proposed solution described in the PR body
- [ ] `yarn check-types` passes
- [ ] `yarn test` passes (add new tests for new behavior)
- [ ] New database migrations included with `up()` and `down()`, if applicable
- [ ] All queries are workspace-scoped (tenant isolation maintained)
- [ ] No secrets, API keys, or customer data included
- [ ] Documentation updated (README, relevant `docs/` file, or inline)
- [ ] PR is focused — large changes split into smaller, reviewable PRs
- [ ] Screenshots included for dashboard UI changes

---

## Adding provider pricing fixtures

Pricing records power the cost engine. Adding or verifying a provider's pricing is one of the most valuable contributions.

**Where:** `packages/cost-engine/src/prices/` (or the equivalent in the current structure).

**Each record includes:**

- `provider` — normalized lowercase string (e.g., `openai`, `anthropic`)
- `model` — the exact model ID as returned by the provider API (e.g., `gpt-4o-2024-08-06`)
- `effectiveFrom` — ISO date when this price became active
- `inputTokenRate` — cost per 1M input tokens in USD
- `outputTokenRate` — cost per 1M output tokens in USD
- `cachedInputTokenRate` — cost per 1M cached input tokens (if applicable)
- `sourceUrl` — link to the provider's pricing page

**Include a test** that asserts `calculateCost()` returns the expected value for a known input, using this record. See existing tests in `packages/cost-engine/src/` for the pattern.

**Source of truth:** Always use the provider's official pricing page. Record the URL and the date you verified the price. Do not guess or interpolate from similar models.

---

## Adding a database migration

1. Create `packages/database/src/migrations/00N_description.ts` (increment the prefix).
2. Export `up(knex: Knex)` and `down(knex: Knex)` functions.
3. `up()` must be backward-compatible with a running v(N-1) API where possible (additive changes preferred).
4. `down()` should reverse the migration cleanly.
5. Test: run `yarn workspace @model-meter/api db:migrate`, verify the schema, then run `yarn workspace @model-meter/api db:rollback` and verify the rollback.

---

## Testing

Model Meter uses [Vitest](https://vitest.dev/).

```bash
# Run all tests
yarn test

# Run tests for one package
yarn workspace @model-meter/cost-engine test
yarn workspace @model-meter/contracts test
yarn workspace @model-meter/sdk test

# Run tests in watch mode
yarn workspace @model-meter/cost-engine test --watch
```

### What to test

- **Cost engine** — fixture tests for every provider/model pricing record. Test edge cases: zero tokens, cached-only usage, unknown model.
- **Contracts** — schema validation for valid and invalid event payloads.
- **SDK** — unit tests for the buffer, flush logic, and OTel exporter.
- **API** — integration tests for ingestion auth, batch validation, and analytics queries.
- **Tenant isolation** — tests that verify a workspace cannot read another workspace's data.

---

## Security requirements

- Do not introduce code that could expose secrets, API keys, or prompt text.
- Do not remove or weaken tenant isolation checks.
- Do not include unvalidated user input in SQL queries — use parameterized queries.
- For auth or RLS changes, add or update cross-tenant isolation tests.
- Report security vulnerabilities privately — see [SECURITY.md](SECURITY.md).

---

## Documentation contributions

Documentation improvements are always welcome:

- Fixing inaccuracies or outdated information
- Adding provider-specific notes or integration examples
- Clarifying architecture decisions
- Translating docs (open an issue first to coordinate)

The documentation lives in `docs/` and the root markdown files. Use clear, direct language. Avoid implying features are available when they are not.

---

## Getting help

- [GitHub Discussions](https://github.com/jayanti-prajapati/model-meter/discussions) — questions and ideas
- [GitHub Issues](https://github.com/jayanti-prajapati/model-meter/issues) — bugs and feature requests
- [SUPPORT.md](SUPPORT.md) — full support options

---

## Good first issues

Browse [`good first issue`](https://github.com/jayanti-prajapati/model-meter/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22) on GitHub for issues tagged as approachable for new contributors.

The best starting points:

- Add or verify a missing provider pricing fixture
- Improve error messages in the ingestion API
- Add tests for an untested code path
- Fix a documentation gap
- Improve dashboard UX (React / shadcn/ui)
