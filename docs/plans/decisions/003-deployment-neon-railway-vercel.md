# ADR-003 — Free deployment stack: Neon + Railway + Vercel

**Status:** ✅ Decided  
**Date:** October 2026

---

## Context

Model Meter is open-source and self-hostable. For early pilots and demos, a zero-cost cloud deployment is needed that matches the existing Docker-based architecture without requiring provider lock-in.

## Decision

| Component | Provider | Reason |
|---|---|---|
| PostgreSQL | [Neon](https://neon.tech) | 0.5 GB free, serverless Postgres, drop-in `DATABASE_URL`, no project pausing |
| API (`apps/api`) | [Railway](https://railway.app) | $5/month free credit, auto-detects monorepo Dockerfile, env vars via UI |
| Admin (`apps/admin`) | [Vercel](https://vercel.com) | Free static hosting, `VITE_API_URL` injected as build env var, global CDN |
| Worker (`apps/worker`) | Skip for now | Events now persist inline in the API (see ADR-002) |

**Total cost: $0** at low traffic.

## Alternatives considered

| Provider | Verdict |
|---|---|
| Supabase (Postgres) | Projects pause after 1 week inactivity on free tier — bad for demos |
| ElephantSQL | 20 MB limit — too small even for seed data |
| Render (API) | Free tier spins down after 15 min idle — cold-start adds ~30 s |
| Fly.io (API) | Good, always-on free tier (3 VMs), more CLI setup required |
| Netlify (admin) | Equivalent to Vercel; either works |
| Cloudflare Pages (admin) | Excellent CDN, slightly more involved build config |

## Required repo changes

### `vercel.json` (root)
```json
{
  "buildCommand": "yarn workspace @model-meter/contracts build && yarn workspace @model-meter/admin build",
  "outputDirectory": "apps/admin/dist",
  "installCommand": "PERSONAL_GITHUB_TOKEN=placeholder yarn install",
  "framework": null
}
```

Set in Vercel UI:
- `VITE_API_URL` → Railway API URL (e.g. `https://model-meter-api.up.railway.app`)
- `VITE_MODEL_METER_API_KEY` → the ingestion key from Neon DB

### Railway (UI config)
Point at `apps/api/Dockerfile` from the monorepo root.

Environment variables:
```
DATABASE_URL      = <Neon pooled connection string>
INGESTION_KEY_SECRET = <random 32+ char secret>
NODE_ENV          = production
PORT              = 3000
```

### Neon (no code changes)
Sign up → create project → copy pooled `DATABASE_URL` → paste into Railway. The API auto-migrates on startup.

## Trade-offs

- **CORS**: The API's `cors()` currently allows all origins. For production, set `CORS_ORIGIN` to the Vercel domain.
- **`VITE_MODEL_METER_API_KEY` is semi-public**: It is an ingestion key sent from the browser. Treat it as a low-privilege credential — rotate if compromised, but it grants only write access to one workspace's events.
- **Railway credit**: $5/month covers ~500 hours of a 512 MB instance. Sufficient for demos; upgrade to paid when traffic warrants.
- **No SLA on free tiers**: Not suitable for production customer data without upgrading.

## When to revisit

When the first paying customer signs up, evaluate:
- Neon Launch plan ($19/month) for more storage and dedicated compute
- Railway Hobby plan ($5/month guaranteed) or a VPS
- Cloudflare Workers for the API if edge latency matters
