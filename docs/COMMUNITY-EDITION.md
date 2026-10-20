# Community Edition

## Mission

Make practical AI usage visibility available to small engineering teams through a free, self-hostable community project.

## Proposed free scope

- Self-hosted web dashboard and API.
- Workspace and project organization.
- SDK and supported gateway ingestion.
- Versioned pricing estimates.
- Spend, request, token, error, and latency dashboards.
- Basic budgets and alerts.
- Billing imports and reconciliation as supported integrations mature.
- Basic offline model evaluation and cost-to-quality comparison.
- Documentation and example integrations.

## Proposed boundaries for a future hosted offering

No hosted/commercial feature split is finalized. Possible hosted value-adds may include managed hosting, longer retention, managed provider integrations, advanced collaboration, SSO, enterprise audit controls, and support. Do not remove basic export or data portability from the community edition without a deliberate product and license decision.

## License decision

Choose and publish the project license before the first public code release. This scaffold includes a placeholder `LICENSE` file that must be replaced with the complete text of the selected license. Possible choices:
- Apache-2.0: permissive, explicit patent grant.
- MIT: simple permissive license.
- AGPL-3.0: copyleft obligations for modified network services.

The right choice depends on whether the project prioritizes permissive adoption or network-copyleft reciprocity. This is a maintainer decision, not legal advice.

## Community governance

- Public roadmap and issue tracker.
- Contributor guide and code of conduct.
- Security vulnerability reporting process.
- Documented decision records for major architecture choices.
- No unannounced collection of telemetry from self-hosted installations.
- No telemetry from customer deployments unless explicitly opt-in and documented.

## Release principles

- Mark planning versus implemented functionality accurately.
- Publish migration notes for schema changes.
- Keep setup reproducible.
- Include tests for tenant isolation and cost calculations.
- Maintain export and deletion capabilities.
