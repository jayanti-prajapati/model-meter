# Plans

This directory is the source of truth for Model Meter's milestones, architecture decisions, and research spikes. Everything here is versioned in git so decisions are reviewable in PRs and visible to the whole team.

## How to use

| Need to... | Do this |
|---|---|
| Plan a new feature or milestone | Create `milestones/M{N}-{slug}.md` |
| Record an architecture decision | Create `decisions/{NNN}-{slug}.md` |
| Document a research investigation | Create `spikes/{slug}.md` |
| Ask Devin to implement a plan | "Implement the plan in docs/plans/milestones/M1-auth-workspace.md" |

---

## Milestones

| File | Title | Status |
|---|---|---|
| [M0-foundation.md](milestones/M0-foundation.md) | Foundation — monorepo, SDK, API, admin | ✅ Shipped |
| [M1-auth-workspace.md](milestones/M1-auth-workspace.md) | Auth & workspace UI | 🔜 Next |
| [M2-gateway.md](milestones/M2-gateway.md) | Gateway adapters | 🔜 Planned |
| [M3-alerts.md](milestones/M3-alerts.md) | Budgets & alerts | 🔜 Planned |
| [M4-billing.md](milestones/M4-billing.md) | Billing reconciliation | 🔜 Planned |
| [M5-quality.md](milestones/M5-quality.md) | Cost-to-quality evaluation | 🔜 Planned |

---

## Architecture Decision Records

| File | Title | Status |
|---|---|---|
| [001-shadcn-over-mui.md](decisions/001-shadcn-over-mui.md) | Replace Material UI with shadcn/ui | ✅ Decided |
| [002-inline-processing-over-queue.md](decisions/002-inline-processing-over-queue.md) | Process events inline in API (not separate worker) | ✅ Decided |
| [003-deployment-neon-railway-vercel.md](decisions/003-deployment-neon-railway-vercel.md) | Free deployment stack: Neon + Railway + Vercel | ✅ Decided |

> Pre-implementation ADRs from the design phase live in [../DECISIONS.md](../DECISIONS.md).

---

## Spikes

| File | Title | Status |
|---|---|---|
| *(none yet)* | | |

---

## Naming conventions

```
milestones/  M{N}-{kebab-slug}.md       e.g. M2-gateway.md
decisions/   {NNN}-{kebab-slug}.md      e.g. 004-sqs-vs-redis.md
spikes/      {kebab-slug}.md            e.g. anthropic-pricing.md
```
