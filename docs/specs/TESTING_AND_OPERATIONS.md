# Archon Testing and Operations Strategy

**Status:** Proposed for approval  
**Last updated:** 2026-07-18

## 1. Test layers

### Unit tests

Cover pure domain rules, state transitions, schema validation, authorization policies, prompt-input construction, consistency rules, renderers, and retry classification. Unit tests must not require PostgreSQL, Redis, network access, or provider credentials.

### Integration tests

Cover Prisma repositories against PostgreSQL, transaction boundaries, outbox dispatch, BullMQ handlers against Redis, authentication/session adapters, API routes, and object-storage adapters. Integration suites must use isolated test data and deterministic cleanup.

### Contract tests

Verify:

- API request/response schemas
- queue payload schemas
- provider adapter behavior using recorded synthetic fixtures or fakes
- artifact schema backward compatibility
- export manifest and Markdown ordering

No test fixture may contain real customer data, live credentials, or unredacted provider payloads.

### Browser acceptance tests

Playwright tests cover:

1. sign in and sign out
2. create workspace-scoped project
3. submit idea and constraints
4. view and answer clarification questions
5. accept or reject assumptions
6. start generation and observe progress
7. recover from a visible generation failure
8. review, edit, regenerate, and approve one artifact
9. view consistency findings
10. export and download Markdown
11. verify a second user cannot access the project
12. archive and restore a project

Critical tests must use the fake provider and deterministic worker execution. A small separately tagged suite may exercise a staging provider account with strict cost limits.

## 2. Security tests

Required automated cases:

- horizontal privilege escalation across workspaces
- forged workspace/project identifiers
- missing and expired sessions
- CSRF on cookie-authenticated mutations
- unsafe Markdown/HTML content in input and model output
- prompt injection strings treated as data rather than instructions to application tooling
- oversized payload and export limits
- secret-redaction tests for logs and errors
- provider error sanitization
- idempotency-key replay and conflict behavior
- SQL injection regression through all filter/search inputs
- rate-limit enforcement

A threat-model review is required before the first production release and after material changes to authentication, authorization, provider integrations, exports, or public exposure.

## 3. AI quality evaluation

Maintain a versioned, synthetic evaluation set representing:

- CRUD SaaS application
- multi-role marketplace
- event-driven integration service
- privacy-sensitive internal tool
- ambiguous early-stage idea
- intentionally conflicting constraints
- prohibited technology requirement
- moderate scale and high scale variants

Evaluation dimensions:

- completeness of required artifacts
- schema-valid output rate
- internal consistency
- traceability to user constraints
- unsupported assumption count
- security-boundary quality
- actionable roadmap quality
- valid Mermaid rate
- human-review severity and rewrite requirement

Prompt or model changes that materially reduce benchmark quality must not ship without documented acceptance.

## 4. Performance targets

Initial service-level objectives for staging and production readiness:

- 95% of ordinary authenticated API reads under 500 ms excluding network edge latency
- generation command acknowledged under 1 second
- job progress visible within 2 seconds of stage change
- project list supports at least 1,000 projects per workspace through pagination
- no unbounded query or artifact list endpoints
- export of the configured maximum package size completes within 30 seconds under target infrastructure

Generation completion time is provider-dependent and is reported as an operational metric rather than a hard user-facing guarantee for MVP.

## 5. CI pipeline

Required pull-request jobs:

1. dependency install with lockfile enforcement
2. formatting check
3. lint
4. TypeScript type-check
5. unit tests with coverage report
6. PostgreSQL integration tests
7. Redis/worker integration tests
8. migration validation
9. Playwright critical-path tests
10. production build
11. dependency vulnerability scan
12. secret scan

Required main-branch jobs:

- all pull-request checks
- build immutable application and worker artifacts
- publish provenance/checksum metadata where supported
- deploy to staging only through an explicit approved workflow
- run smoke tests after deployment

Production deployment must require explicit approval and must not occur directly from an unreviewed branch.

## 6. Coverage policy

Coverage percentage is a signal, not the acceptance criterion. The following behavior requires direct tests regardless of aggregate percentage:

- authorization policies
- project and revision state transitions
- transaction/outbox behavior
- idempotency
- retry classification
- schema and consistency validation
- approval semantics
- export manifest selection
- retention and deletion
- sanitization and secret redaction

## 7. Environments

### Local

- Docker Compose PostgreSQL and Redis
- fake provider by default
- optional developer-owned provider key outside source control
- seeded synthetic test account and projects

### CI

- ephemeral PostgreSQL and Redis
- fake provider only for required checks
- no production secrets

### Staging

- production-like authentication, database, queue, worker, storage, logging, and telemetry
- isolated provider project and strict budgets
- synthetic data only unless separately approved
- automatic retention shorter than production

### Production

- private network boundaries where supported
- managed PostgreSQL and Redis with backups
- secret manager
- protected object storage
- centralized logs, metrics, traces, and alerts
- least-privilege deployment identity

The exact hosting provider is intentionally not selected by this specification.

## 8. Deployment design

Deploy the web/API process and generation worker as independently scalable artifacts from the same versioned source commit.

Release metadata must include:

- Git commit SHA
- database schema version
- artifact and prompt schema versions
- build timestamp
- application version

Deployment order:

1. validate backward-compatible migration
2. apply migration using a dedicated job
3. deploy web/API
4. deploy worker
5. run smoke tests
6. monitor errors, queue health, and provider failures

Breaking migrations require an expand-and-contract approach. Application rollback must remain possible while both old and new versions may temporarily access the schema.

## 9. Backups and recovery

- PostgreSQL automated backups and point-in-time recovery where supported
- object-storage versioning or equivalent protection for export bundles during their retention window
- Redis is not the source of truth; recoverable jobs must be reconstructable from database state/outbox records
- quarterly restore test before production maturity, with at least one successful restore before initial release

Recovery documentation must define owners, commands, expected recovery point, and validation steps.

## 10. Operational runbooks

Required runbooks:

- provider outage or elevated refusal/error rate
- stuck or growing queue
- failed database migration
- suspected cross-tenant authorization incident
- leaked or rotated credential
- object-storage failure
- excessive generation cost
- corrupted or invalid artifact output
- deletion/retention job failure

Each runbook must state detection, immediate containment, diagnosis, recovery, communication, and follow-up actions.

## 11. Alerts

Minimum alerts:

- API 5xx rate above threshold
- authentication failure anomaly
- authorization-denial anomaly
- queue oldest-job age above threshold
- job failure/retry rate above threshold
- database or Redis unavailable
- provider throttling or error spike
- cost budget threshold reached
- export error spike
- retention worker failure

Alerts must link to the relevant dashboard and runbook.

## 12. Release acceptance evidence

The release PR or release record must link to:

- green CI run
- migration verification
- authorization and tenant-isolation results
- critical Playwright results
- AI quality benchmark summary
- accessibility report
- security review/threat model
- staging smoke test
- rollback plan
- known risks and accepted exceptions

No task or milestone is `completed` until its required evidence exists and all blocking checks pass.