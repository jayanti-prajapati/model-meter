# Model Meter — Developer Reference

## Prerequisites

- Node.js 22 LTS
- Corepack / Yarn 4: `corepack enable`
- Docker (for local Postgres and full stack)

## Quick start (local dev)

```bash
# 1. Install dependencies
yarn install

# 2. Start Postgres
docker compose -f docker/docker-compose.yml up postgres -d

# 3. Copy env and configure
cp .env.example .env   # Fill in DATABASE_URL etc.

# 4. Start API (auto-migrates on start)
yarn workspace @model-meter/api dev

# 5. Start admin UI (separate terminal)
yarn workspace @model-meter/admin dev

# 6. Start worker (separate terminal)
yarn workspace @model-meter/worker dev
```

Admin UI: http://localhost:5173
API: http://localhost:3000

## Common commands

| Command | Description |
|---|---|
| `yarn build` | Build all packages (via Turborepo) |
| `yarn check-types` | Type-check all packages |
| `yarn test` | Run all tests |
| `yarn turbo run dev` | Start all dev servers concurrently |
| `yarn workspace @model-meter/api db:migrate` | Run DB migrations manually |

## Full stack with Docker

```bash
cp .env.example .env
# Edit .env — set INGESTION_KEY_SECRET to a random 32-char secret
docker compose -f docker/docker-compose.yml up -d
```

- Admin UI: http://localhost:5173
- API health: http://localhost:3000/health

## Package structure

| Path | Package name | Description |
|---|---|---|
| `packages/typescript-config` | `@model-meter/typescript-config` | Shared tsconfig presets |
| `packages/contracts` | `@model-meter/contracts` | Zod schemas + TypeScript types |
| `packages/database` | `@model-meter/database` | Knex client, migrations, repositories |
| `packages/cost-engine` | `@model-meter/cost-engine` | Pure cost calculation |
| `packages/sdk-node` | `@model-meter/sdk` | npm SDK for customer apps |
| `apps/api` | `@model-meter/api` | Express API server |
| `apps/worker` | `@model-meter/worker` | Background event processor |
| `apps/admin` | `@model-meter/admin` | React/Vite dashboard |

## SDK usage (in a customer app)

```bash
npm install @model-meter/sdk
```

```ts
import { init } from '@model-meter/sdk';

// Call once at app startup
init({
  apiKey:      process.env.MODEL_METER_API_KEY,
  endpoint:    process.env.MODEL_METER_ENDPOINT,
  projectId:   'my-project',
  environment: 'production',
});
```

Auto-tracking via OTel (zero code changes needed after `init()`):
```ts
import { registerTelemetry } from 'ai';
import { OpenTelemetry } from '@ai-sdk/otel';
import { createModelMeterExporter } from '@model-meter/sdk';
import { NodeTracerProvider } from '@opentelemetry/sdk-trace-node';
import { SimpleSpanProcessor } from '@opentelemetry/sdk-trace-base';
import { getClient } from '@model-meter/sdk';

const provider = new NodeTracerProvider({
  spanProcessors: [
    new SimpleSpanProcessor(createModelMeterExporter(getClient()!)),
  ],
});
provider.register();
registerTelemetry(new OpenTelemetry());
```

Optional wrapper (adds operation label):
```ts
import { generateText } from '@model-meter/sdk';

const result = await generateText({
  model: openai('gpt-4o'),
  prompt: 'Summarize this ticket',
  modelMeter: { operation: 'summarize-ticket' },
});
```

## Adding a new migration

1. Create `packages/database/src/migrations/00N_description.ts`
2. Export `up(knex)` and `down(knex)` functions
3. The API auto-runs pending migrations on next startup

## Test commands

```bash
# Run all tests
yarn test

# Run tests for a specific package
yarn workspace @model-meter/cost-engine test
yarn workspace @model-meter/contracts test
yarn workspace @model-meter/sdk test
```
