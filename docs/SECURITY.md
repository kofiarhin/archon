# Archon Security Baseline

**Status:** Proposed  
**Last updated:** 2026-07-18

## Security objectives

- protect project data and generated artifacts from unauthorized access
- prevent secrets from entering prompts, logs, exports, or source control
- treat user input and AI output as untrusted
- preserve tenant isolation across all project operations
- make security-sensitive actions traceable

## Required controls

### Authentication and authorization

- Require authenticated access for private project data.
- Enforce workspace and project authorization server-side on every operation.
- Use least-privilege roles and deny access by default.
- Test direct-object-reference and cross-tenant access attempts.

### AI and prompt security

- Do not send application secrets or provider credentials in prompts.
- Delimit user-provided content from system instructions.
- Validate structured outputs against strict schemas.
- Do not execute generated commands, migrations, or infrastructure changes automatically.
- Treat imported content as potentially containing prompt injection.

### Data protection

- Use TLS in transit and provider-managed encryption at rest.
- Define retention and deletion policies before production launch.
- Avoid collecting unnecessary personal data.
- Redact sensitive values from logs and error reporting.

### Application security

- Validate all inputs at API boundaries.
- Sanitize or safely render Markdown, Mermaid, and exported content.
- Apply rate limits to authentication and generation endpoints.
- Protect state-changing browser requests against CSRF where applicable.
- Use secure cookie and session settings when cookie-based authentication is used.

### Supply chain and operations

- Pin dependencies through a lockfile.
- Run dependency, secret, lint, type, test, and build checks in CI.
- Keep production credentials outside the repository.
- Rotate compromised credentials immediately.
- Record provider and model usage metadata without recording prompt secrets.

## Security acceptance criteria

Before the MVP is considered production-ready:

- cross-project and cross-workspace authorization tests pass
- no secrets are present in source control or generated exports
- project deletion and retention behavior is documented and tested
- rate limiting and abuse controls are enabled
- stored and rendered AI output is handled as untrusted content
- dependency and secret scanning run in CI
- a basic threat model has been reviewed

## Incident handling

Security incidents must be documented with impact, affected data, containment, remediation, and follow-up actions. Confirmed exposed credentials must be revoked rather than merely removed from Git history.
