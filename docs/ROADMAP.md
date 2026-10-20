# Roadmap and Validation Plan

## Product validation before feature expansion

Interview 10–15 CTOs, engineering managers, and AI engineers at small software companies. Ask them to demonstrate their existing workflow, recent unexpected bills, project attribution, current tools, and purchasing process.

Evidence of a real problem:
- They can show a recent costly incident or time-consuming workaround.
- They have tried existing tools or built internal scripts.
- They agree to share safe sample usage data.
- A budget owner agrees to a paid pilot with success criteria.

Polite interest or free sign-ups alone are weak evidence.

## Milestone 1 — Foundation and tenancy

- Monorepo, linting, tests, environment validation.
- Workspace, user, membership, project, integration schema.
- Authentication and role enforcement.
- RLS policies and cross-tenant tests.

**Exit:** Cross-workspace access tests pass for reads, writes, exports, and mutations.

## Milestone 2 — Ingestion

- Versioned event contract.
- SDK and batch ingestion endpoint.
- One gateway adapter.
- Idempotency and queue worker.
- Error tracking and dead-letter queue.

**Exit:** Retries do not duplicate usage, and telemetry failures are observable.

## Milestone 3 — Cost engine and dashboard

- Effective-dated pricing.
- Daily aggregates.
- Summary, time series, provider/model/project breakdown.
- Clear estimated-cost provenance.

**Exit:** Results match deterministic fixtures and known pricing cases.

## Milestone 4 — Budgets and sharing

- Workspace/project budgets.
- Threshold alerts.
- Member invitations and roles.
- Authenticated read-only report sharing.

**Exit:** Alerts are delivered once per threshold-crossing policy and permissions are enforced.

## Milestone 5 — Billing reconciliation

- One provider billing import.
- Checkpointed, paginated sync.
- Billing records and reconciliation report.
- Late adjustment handling.

**Exit:** Estimated and provider-reported totals remain separate and explainable.

## Milestone 6 — Cost-to-quality

- One representative task.
- Versioned evaluation dataset.
- Candidate model comparison.
- Cost per successful task and latency comparison.
- Human-approved recommendations.

**Exit:** Recommendation includes evidence, constraints, and limitations.

## Pilot metrics

Target for first validation cycle:
- 10–15 completed interviews.
- At least 5 prospects describing the same recurring, costly problem.
- 3 paid pilot customers.
- At least 2 customers using the dashboard weekly.
- Evidence of useful cost findings or reduced analysis time.
- A plausible renewal price and hosting/support cost model.

These are go/no-go targets, not predicted results. If existing tools already solve the problem adequately, narrow the segment or change the product hypothesis.

## Initial pricing hypotheses (if a hosted plan is later offered)

Potential test points: $29/month starter, $99/month team, and $249/month business. These are experiments, not established market prices. The community edition is intended to remain free and self-hostable.
