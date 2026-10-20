# Getting Started with Model Meter

This guide walks you from zero to a working Model Meter installation with live AI cost tracking.

**Time to complete:** 15–30 minutes

---

## Prerequisites

- **Docker and Docker Compose** — for the quickstart
- **Node.js 22+** and **Yarn 4** (via Corepack) — for local development without Docker
- An application that makes LLM API calls (OpenAI, Anthropic, etc.) — to send test events

---

## Option A: Docker Quickstart (recommended)

The fastest path to a running Model Meter stack.

### Step 1: Clone and configure

```bash
git clone https://github.com/jayanti-prajapati/model-meter.git
cd model-meter

cp .env.example .env
```

Open `.env` and set `INGESTION_KEY_SECRET` to a random 32-character string:

```bash
# Generate a secret:
openssl rand -hex 32
```

Paste the output as the value of `INGESTION_KEY_SECRET` in your `.env` file.

### Step 2: Start the stack

```bash
docker compose -f docker/docker-compose.yml up -d
```

The API migrates the database automatically on first startup. Wait about 10 seconds, then verify:

```bash
curl http://localhost:3000/health
# Expected: {"status":"ok","db":"ok"}
```

| Service | URL |
|---|---|
| Admin dashboard | http://localhost:5173 |
| API health check | http://localhost:3000/health |

### Step 3: Seed a workspace and ingestion key

Until the auth UI ships (Milestone 1), you need to manually seed an ingestion key.

Connect to Postgres:

```bash
docker compose -f docker/docker-compose.yml exec postgres \
  psql -U postgres -d model_meter
```

Then run:

```sql
-- 1. Create a workspace
INSERT INTO workspaces (id, name, created_at, updated_at)
VALUES (gen_random_uuid(), 'My Team', now(), now())
RETURNING id;
-- Note the workspace_id

-- 2. Create a project (replace <workspace_id> with the UUID from step 1)
INSERT INTO projects (id, workspace_id, name, created_at, updated_at)
VALUES (gen_random_uuid(), '<workspace_id>', 'My App', now(), now())
RETURNING id;
-- Note the project_id

-- 3. Create an integration
INSERT INTO integrations (id, workspace_id, project_id, name, source, created_at, updated_at)
VALUES (gen_random_uuid(), '<workspace_id>', '<project_id>', 'SDK', 'sdk', now(), now())
RETURNING id;
-- Note the integration_id

-- 4. Create an ingestion key (replace secrets with your values)
INSERT INTO ingestion_keys (id, workspace_id, integration_id, key_hash, label, created_at, updated_at)
VALUES (
  gen_random_uuid(),
  '<workspace_id>',
  '<integration_id>',
  encode(sha256('my-secret-key-123'::bytea), 'hex'),
  'dev-key',
  now(),
  now()
)
RETURNING id;
```

Your ingestion key value is `my-secret-key-123` (the plaintext; only the hash is stored). Use this as `MODEL_METER_API_KEY` in your application.

---

## Option B: Local Development (no Docker for app code)

Use this if you want to run the API and admin locally while editing code.

### Step 1: Install dependencies

```bash
# Enable Corepack for Yarn 4
corepack enable

# Install all workspace dependencies
yarn install
```

### Step 2: Start Postgres

```bash
docker compose -f docker/docker-compose.yml up postgres -d
```

### Step 3: Configure environment

```bash
cp .env.example .env
# Edit .env:
#   DATABASE_URL=postgresql://postgres:postgres@localhost:5432/model_meter
#   INGESTION_KEY_SECRET=<your-random-32-char-secret>
```

### Step 4: Start services

In separate terminal windows:

```bash
# Terminal 1 — API (auto-migrates on start)
yarn workspace @model-meter/api dev

# Terminal 2 — Admin UI
yarn workspace @model-meter/admin dev
```

The API starts on port 3000; the admin UI on port 5173.

---

## Instrument Your Application

### Step 1: Install the SDK

```bash
npm install @model-meter/sdk
# or
yarn add @model-meter/sdk
```

### Step 2: Initialize at app startup

Create or edit your instrumentation file (e.g., `instrumentation.ts`):

```ts
import { init } from '@model-meter/sdk';

init({
  apiKey:      'my-secret-key-123',     // your ingestion key plaintext
  endpoint:    'http://localhost:3000',  // your Model Meter API URL
  projectId:   'my-app',                // matches the project name you created
  environment: 'development',
});
```

Call `init()` **once**, as early as possible — before any AI calls are made.

### Step 3: Auto-track via OpenTelemetry (recommended)

If you use the [Vercel AI SDK](https://sdk.vercel.ai/), add the OTel span processor:

```ts
import { registerTelemetry } from 'ai';
import { OpenTelemetry } from '@ai-sdk/otel';
import { createModelMeterExporter, getClient } from '@model-meter/sdk';
import { NodeTracerProvider } from '@opentelemetry/sdk-trace-node';
import { SimpleSpanProcessor } from '@opentelemetry/sdk-trace-base';

const provider = new NodeTracerProvider({
  spanProcessors: [
    new SimpleSpanProcessor(createModelMeterExporter(getClient()!)),
  ],
});
provider.register();
registerTelemetry(new OpenTelemetry());
```

After this, every `generateText()`, `streamText()`, `generateObject()`, or `embed()` call is tracked automatically.

### Step 4: Make a test AI call

```ts
import { generateText } from 'ai';
import { openai } from '@ai-sdk/openai';

const result = await generateText({
  model: openai('gpt-4o-mini'),
  prompt: 'Hello, Model Meter!',
});

console.log(result.text);
```

Events are buffered and flushed every 5 seconds. After a few seconds, open the admin dashboard at http://localhost:5173 — you should see the usage event appear.

### Step 5: Add operation labels (optional)

Use the `@model-meter/sdk` wrapper to label calls by operation for more granular cost attribution:

```ts
import { generateText } from '@model-meter/sdk'; // drop-in for 'ai'

const result = await generateText({
  model: openai('gpt-4o'),
  prompt: 'Summarise this support ticket...',
  modelMeter: { operation: 'summarise-ticket' },
});
```

Operations appear as a breakdown dimension in the dashboard.

---

## Verify Data is Flowing

1. Open the admin at http://localhost:5173
2. Navigate to **Usage Events** — your test event should appear within 10 seconds
3. Navigate to **Dashboard** — spend totals update after the first event

If no events appear:
- Check `curl http://localhost:3000/health` returns `{"status":"ok","db":"ok"}`
- Check API logs: `docker compose -f docker/docker-compose.yml logs api`
- Ensure your `apiKey` in `init()` matches the plaintext key you seeded

---

## Serverless and Edge Runtimes

For serverless functions (AWS Lambda, Vercel Functions), buffered events may not flush before the function exits. Call `flush()` explicitly before returning:

```ts
import { flush } from '@model-meter/sdk';

export async function handler(event) {
  // ... your AI call ...
  await flush(); // ensure events are sent before the function exits
  return response;
}
```

---

## Next Steps

- [SDK Reference](INGESTION.md) — full event schema and SDK API
- [API Design](API.md) — available endpoints and request formats
- [Architecture](ARCHITECTURE.md) — how the components fit together
- [Deployment](DEPLOYMENT.md) — production deployment on Railway, AWS, or Docker
- [FAQ](FAQ.md) — common questions answered
