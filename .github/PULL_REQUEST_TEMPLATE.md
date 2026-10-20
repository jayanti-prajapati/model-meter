## Summary

<!-- Describe what this PR does and why. Link the related issue if applicable. -->

Closes #

## Changes

<!-- Bullet-point list of what changed. -->

- 

## Type of change

- [ ] Bug fix
- [ ] New feature
- [ ] Refactor (no behavior change)
- [ ] Documentation
- [ ] Dependency update
- [ ] CI / infrastructure

## Testing

<!-- Describe how you tested this. Include commands to run tests. -->

```bash
yarn test
```

- [ ] Existing tests pass (`yarn test`)
- [ ] New tests added for new behavior
- [ ] Tested locally with Docker Compose

## Checklist

- [ ] Code follows the TypeScript conventions in this repo
- [ ] Inputs validated with Zod schemas where applicable
- [ ] All queries are workspace-scoped (tenant isolation maintained)
- [ ] No secrets, API keys, or customer data included
- [ ] Database migration included if schema changed, with `up()` and `down()`
- [ ] Documentation updated (README, relevant `docs/` file, or inline comments)
- [ ] `yarn check-types` passes
- [ ] PR is focused and reviewable — large changes split into smaller PRs

## Security impact

<!-- Does this change affect authentication, authorization, ingestion key handling, tenant isolation, or data access? If yes, describe. -->

None / see below:

## Migration notes

<!-- If this PR includes a database migration, describe what it does and any backwards-compatibility considerations. -->

N/A
