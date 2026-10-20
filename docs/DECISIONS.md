# Architecture Decision Records

## ADR-001: Start with a modular monolith

**Status:** Proposed

**Decision:** Build web, API, and worker as separately deployable processes in one TypeScript monorepo, sharing packages.

**Reason:** Faster development, simpler local setup, shared contracts, and fewer operational systems for an early product.

**Revisit when:** Teams, scaling requirements, security boundaries, or release cadence justify service separation.

## ADR-002: Hybrid usage collection

**Status:** Proposed

**Decision:** Support gateway events, a TypeScript SDK, and provider billing imports. Normalize into one canonical event contract while retaining source provenance.

**Reason:** No single collection method captures every relevant kind of usage and billing information.

**Trade-off:** More complex deduplication and reconciliation. The product must distinguish estimated request costs from provider-billed amounts.

## ADR-003: PostgreSQL as initial source of truth

**Status:** Proposed

**Decision:** Use PostgreSQL for operational data and dashboard aggregates in the MVP.

**Reason:** Strong relational constraints, transactional writes, JSON support, and mature AWS hosting.

**Revisit when:** Measured event volume or analytics queries exceed PostgreSQL's practical limits.

## ADR-004: Privacy-first telemetry

**Status:** Proposed

**Decision:** Do not capture prompts/completions by default.

**Reason:** Minimize risk and reduce collection of customer secrets and personal data.

## ADR-005: Recommendations before automatic routing

**Status:** Proposed

**Decision:** Begin with offline model evaluations and human-approved recommendations.

**Reason:** Model quality is task-specific; automatic switching before trustworthy evaluations could degrade customer outcomes.
