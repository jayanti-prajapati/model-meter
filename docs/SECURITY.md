# Security and Tenant Isolation

Security is a product requirement, not a later enhancement. Model Meter may process commercial usage data and provider credentials.

## Tenant isolation

- Authenticate every dashboard request.
- Derive user identity from a verified session/token.
- Check membership and role for every workspace-scoped action.
- Derive ingestion scope from the credential, not the request body.
- Include `workspace_id` in all tenant-owned tables and queries.
- Use PostgreSQL Row-Level Security (RLS) as defense in depth.
- Apply RLS to all tenant-owned tables, not only usage events.
- Keep the runtime DB role non-superuser and without `BYPASSRLS`.
- Use a separate migration role. Force RLS where appropriate.
- Test cross-tenant read, write, update, export, and deletion attempts.

Illustrative policy:

```sql
ALTER TABLE usage_events ENABLE ROW LEVEL SECURITY;
ALTER TABLE usage_events FORCE ROW LEVEL SECURITY;

CREATE POLICY usage_events_workspace_policy
ON usage_events
USING (
  workspace_id =
    NULLIF(current_setting('app.workspace_id', true), '')::uuid
)
WITH CHECK (
  workspace_id =
    NULLIF(current_setting('app.workspace_id', true), '')::uuid
);
```

Set the workspace context inside each database transaction only after the application has authorized the request. Use transaction-local settings to avoid leaking context through pooled connections. RLS is not authentication: never allow untrusted callers to set arbitrary database context.

## Credential management

- Store only hashes of customer ingestion keys.
- Support revocation, rotation, scopes, and last-used metadata.
- Store provider API credentials in AWS Secrets Manager or an equivalent secret manager.
- Use IAM task roles; no hardcoded AWS keys.
- Never send provider secrets to the browser or include them in logs.

## Data minimization

- Do not collect prompts or completions by default.
- Avoid collecting personal data in operation names or metadata.
- Redact sensitive headers and error payloads.
- Provide configurable retention and deletion workflows.
- Document exactly which fields the SDK transmits.

## Application security

- TLS in transit and encryption at rest.
- Request schema validation, bounded payloads, rate limits, and abuse monitoring.
- CSRF protection if cookie-based sessions are used.
- Secure cookies, appropriate CORS, and content security policy.
- Dependency scanning, secret scanning, and container image scanning.
- Audit logs for member, integration, budget, and credential changes.
- Tested backup restoration and incident response procedures.

## Telemetry resilience

Customer AI calls should not fail because Model Meter is unavailable. The SDK should use bounded buffering, short timeouts, and controlled retries. Clearly document possible telemetry loss when a process exits before buffered events are durably delivered.

## Reporting a vulnerability

Do not publish exploitable details in a public issue. Until a private security contact is configured, contact the maintainers through GitHub's private vulnerability reporting feature if enabled.
