# Contributing to Model Meter

Thanks for considering a contribution.

## Before you start

- Read the README and relevant architecture docs.
- Search existing issues before opening a new one.
- For substantial changes, open a proposal first so the API, schema, and scope can be discussed.
- Never include real provider keys, customer data, prompts, or private billing exports in issues or pull requests.

## Development principles

- TypeScript for application code.
- Validate external inputs with shared schemas.
- Keep cost calculations deterministic and independently tested.
- Preserve tenant boundaries in every repository and query.
- Make ingestion idempotent.
- Avoid prompt/completion capture by default.
- Document provider-specific behavior and missing fields.
- Mark planned functionality as planned; do not imply integrations are complete when they are not.

## Pull requests

Include:
- Problem and proposed solution.
- Tests and how to run them.
- Database migration notes, if applicable.
- Security and tenant-isolation impact.
- Documentation updates.
- Screenshots for dashboard changes when useful.

## Commit hygiene

Do not commit `.env`, secrets, generated billing files, build outputs, or local databases. Keep pull requests focused and reviewable.
