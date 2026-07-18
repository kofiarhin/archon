# Archon MVP Implementation Specification

**Status:** Proposed for approval  
**Last updated:** 2026-07-18  
**Depends on:** `docs/PRD.md`, `docs/ARCHITECTURE.md`, `docs/SECURITY.md`, `docs/adr/0001-modular-monolith-with-worker.md`

## 1. Purpose

This specification translates the approved Archon MVP product requirements into an implementation-ready technical design. It defines system boundaries, responsibilities, state transitions, storage rules, API behavior, AI orchestration, security controls, observability, and acceptance requirements.

Implementation must not expand the MVP into full application-code generation, autonomous deployment, infrastructure provisioning, billing, enterprise administration, or real-time collaboration.

## 2. Recommended technical decisions

The following choices are proposed and require approval through this pull request:

- **Web:** Next.js with React and TypeScript.
- **API:** Node.js and TypeScript, implemented as server-side application modules within the Next.js deployment boundary where practical, with a separately runnable worker process.
- **Validation:** Zod schemas shared across the web, API, worker, and tests.
- **Database:** PostgreSQL with Prisma ORM and migrations.
- **Queue:** BullMQ backed by Redis.
- **Authentication:** Auth.js with email magic-link and GitHub OAuth for the MVP.
- **Authorization:** server-side workspace membership checks on every project-scoped operation.
- **AI:** OpenAI Responses API through an internal provider adapter; model identifiers are configuration, not domain logic.
- **Storage:** PostgreSQL for project artifacts and metadata; S3-compatible object storage only for generated export bundles above the configured inline-size threshold.
- **Testing:** Vitest for unit/integration tests and Playwright for browser acceptance tests.
- **Local development:** Docker Compose for PostgreSQL and Redis.
- **CI:** GitHub Actions running formatting, lint, type-check, tests, migration validation, and production build.

## 3. Repository structure

```text
apps/
  web/                 # Next.js application and HTTP API routes
  worker/              # generation and export workers
packages/
  domain/              # domain entities, policies, and state transitions
  schemas/             # versioned input and artifact schemas
  ai/                  # provider adapter, prompts, parsers, usage metadata
  database/            # Prisma schema, migrations, repositories
  queue/               # job contracts, queue client, retry policy
  observability/       # logging, metrics, tracing helpers
  config/              # validated runtime configuration
  test-support/        # fixtures, factories, fake provider, utilities
docs/
  adr/
  specs/
```

Import direction must flow from applications into packages. Domain and schema packages must not import application code or provider SDKs.

## 4. Domain model

### 4.1 User

Represents an authenticated person. Identity-provider data is kept separate from project-domain data.

### 4.2 Workspace

The authorization boundary. Every project belongs to exactly one workspace. MVP roles:

- `owner`
- `member`

Only owners may archive a workspace or change membership. Both roles may create and edit projects unless later restricted.

### 4.3 Project

Represents one architecture engagement. States:

- `draft`
- `clarification_required`
- `ready_for_generation`
- `generating`
- `review`
- `approved`
- `archived`

State changes must be performed through domain commands and recorded in an audit event.

### 4.4 Project input

An immutable snapshot containing the idea description, users, workflows, constraints, scale expectations, technology preferences, and accepted assumptions. Editing creates a new input version.

### 4.5 Artifact

A stable logical artifact slot such as `product_summary`, `requirements`, `technology_stack`, `system_architecture`, `data_model`, `api_specification`, `security`, `repository_structure`, `roadmap`, `decision_records`, or `coding_agent_prompts`.

### 4.6 Artifact revision

An immutable version of an artifact. Revision states:

- `generated`
- `edited`
- `validated`
- `proposed`
- `approved`
- `superseded`
- `rejected`

Approval applies to a specific revision and must never be copied automatically to regenerated content.

### 4.7 Decision, assumption, and open question

These are first-class records rather than embedded prose only. Each record stores status, source, owning artifact, created revision, resolution details, and audit timestamps.

### 4.8 Generation job

A retryable unit of work linked to an exact project-input version, schema version, provider configuration, and requested artifact set. States:

- `queued`
- `planning`
- `awaiting_clarification`
- `generating`
- `validating`
- `synthesizing`
- `completed`
- `failed`
- `cancelled`

### 4.9 Export

A reproducible snapshot of selected approved and proposed artifact revisions, including metadata, unresolved questions, assumptions, and Mermaid source.

## 5. Core workflows

### 5.1 Create project

1. Authenticate the user.
2. Resolve the active workspace.
3. Validate input.
4. Create the project and input version in one transaction.
5. Record an audit event.
6. Return the project resource.

### 5.2 Analyze for clarification

1. Create an idempotent planning job.
2. Worker loads the exact input version.
3. Provider returns schema-constrained planning output.
4. Validate schema and run deterministic checks.
5. Persist questions and assumptions transactionally.
6. Set project to `clarification_required` or `ready_for_generation`.

### 5.3 Generate package

1. Reject generation while material questions remain unanswered unless the user explicitly accepts documented assumptions.
2. Create one parent generation job and deterministic child tasks per artifact.
3. Generate artifacts from a shared project context and approved constraints.
4. Validate each artifact before persistence.
5. Run cross-artifact consistency checks.
6. Persist immutable revisions.
7. Move the project to `review` only when the required artifact set exists and validation passes.

### 5.4 Revise one artifact

Manual edits and regeneration both create new revisions. Unrelated artifact revisions remain unchanged. Regeneration receives the current approved constraints and relevant approved artifact summaries, but not hidden secrets or unrelated raw logs.

### 5.5 Approve a revision

Approval requires an explicit authenticated command. The service verifies workspace access, confirms the revision is current and valid, marks any previous approved revision as superseded, and records the actor and timestamp.

### 5.6 Export

The export service selects revisions using an explicit manifest, renders deterministic Markdown, validates Mermaid code blocks, includes approval state and unresolved items, and stores a digest for reproducibility.

## 6. AI orchestration

### 6.1 Pipeline stages

- **Planning:** identify missing information, assumptions, entities, roles, constraints, and candidate components.
- **Generation:** produce individual artifacts using versioned schemas.
- **Validation:** apply schema validation and deterministic domain checks.
- **Consistency:** compare named entities, roles, components, endpoints, technology choices, and security claims across artifacts.
- **Synthesis:** create cross-references and package metadata without rewriting approved content.

### 6.2 Provider adapter contract

The domain calls an internal interface with:

- request type and schema version
- model configuration key
- structured input payload
- timeout and retry policy
- correlation and job identifiers

The adapter returns:

- validated structured output or typed failure
- provider and model identifiers
- latency
- token usage where available
- provider request identifier
- safety or refusal metadata

Provider SDK objects must not leak into domain or persistence interfaces.

### 6.3 Prompt and schema versioning

Every generation stores prompt-template version, schema version, model key, and input digest. Prompt changes require tests and a version increment when they can change persisted output shape or meaning.

### 6.4 Validation rules

At minimum, validation must detect:

- missing required artifacts or sections
- unknown artifact types or schema versions
- entity names referenced but never defined
- API resources without a corresponding data or domain concept
- roles used inconsistently across authorization sections
- contradictory technology choices
- duplicate endpoint method/path pairs
- invalid Mermaid fences or unsupported diagram syntax
- approved decisions contradicted by regenerated content

Failures are stored as structured findings and surfaced to the user; they are never silently repaired after approval.

## 7. API behavior

The API is JSON over HTTPS under `/api/v1`. All mutating requests require authentication, authorization, input validation, CSRF protection where browser cookies are used, and an idempotency key for generation/export commands.

Response envelope:

```json
{
  "data": {},
  "meta": { "requestId": "..." }
}
```

Error envelope:

```json
{
  "error": {
    "code": "PROJECT_NOT_FOUND",
    "message": "Project not found",
    "details": []
  },
  "meta": { "requestId": "..." }
}
```

Errors must use stable machine-readable codes and must not disclose provider prompts, credentials, database details, or cross-tenant identifiers.

## 8. Persistence and transactions

- Workspace identifiers must be present on every workspace-owned table or derivable through a required foreign key.
- Repository methods must scope reads and writes by workspace.
- Immutable revisions are inserted, never updated in place except for narrowly defined processing metadata before publication.
- Project state changes and audit entries occur in the same transaction.
- Job enqueueing uses an outbox record committed with the initiating transaction; a dispatcher publishes outbox events to BullMQ.
- Consumers must be idempotent using stable job keys and uniqueness constraints.
- Soft deletion is allowed only for user-facing recovery. Retention workers must support permanent deletion when policy is approved.

## 9. Background jobs

Queue names:

- `project-planning`
- `artifact-generation`
- `consistency-validation`
- `export-rendering`
- `maintenance`

Retry defaults:

- transient provider/network failures: exponential backoff, maximum 3 attempts
- rate limits: provider-directed delay where available
- schema failures: no blind retry with the same prompt and model; route to repair logic or fail visibly
- authorization and validation failures: never retry

Workers must support graceful shutdown, bounded concurrency, per-workspace rate limiting, job cancellation, and stale-job reconciliation.

## 10. Security and privacy

- Deny access by default and authorize server-side on every project resource.
- Never trust workspace or user identifiers supplied by the browser without membership verification.
- Store secrets only in the runtime secret manager or protected environment variables.
- Redact secrets and sensitive tokens from logs and traces.
- Treat user input and AI output as untrusted content.
- Escape rendered Markdown/HTML and prohibit active content in previews and exports.
- Apply request-size, generation-rate, and export-size limits.
- Use parameterized database access through Prisma.
- Record security-relevant audit events without storing prompt bodies in the audit log.
- Provide project deletion and export deletion commands; final retention durations remain proposed until approved.

Proposed retention defaults:

- application logs: 30 days
- generation metadata: life of project plus 30 days
- failed raw provider payloads: not stored by default
- deleted project recovery window: 14 days
- audit events: 1 year

Projects are private to their workspace in the MVP. Public sharing is out of scope.

## 11. Observability

Every HTTP request and job has a correlation ID. Structured logs include service, environment, request/job ID, workspace-safe hash, project-safe hash, operation, duration, result, and error code.

Required metrics:

- request count, latency, and error rate
- queue depth and oldest-job age
- job duration, retries, and failures by stage
- provider latency, token usage, and estimated cost
- schema and consistency failure counts
- export duration and size
- authorization-denial count

Alerts should cover sustained API failures, stuck queues, high job-failure rates, repeated provider throttling, and database/Redis unavailability.

## 12. Accessibility and UX requirements

- Core flows must be operable by keyboard.
- Forms require explicit labels, inline validation, and error summaries.
- Job progress must use text status as well as visual indicators.
- Regeneration and approval actions require clear scope and consequences.
- Destructive actions require confirmation and communicate retention behavior.
- The production target is WCAG 2.2 AA.

## 13. Configuration

Runtime configuration is validated at startup. Required categories:

- application URL and environment
- database URL
- Redis URL
- authentication secrets and provider credentials
- AI provider API key and model configuration keys
- object-storage settings when enabled
- logging and telemetry endpoints
- rate limits and maximum input/export sizes

Production startup must fail closed when required settings are missing or malformed.

## 14. Definition of done

A feature is complete only when:

- behavior matches this specification and the PRD
- authorization is enforced and tested
- schemas and migrations are committed
- unit and integration tests pass
- browser acceptance coverage exists for critical user behavior
- logs and metrics support diagnosis
- accessibility checks pass for affected flows
- documentation and operational notes are updated
- no secrets or sensitive payloads are committed or logged

## 15. Open approval items

Approval of this specification approves these proposed MVP decisions:

1. Next.js/TypeScript modular monolith plus separate worker.
2. Prisma/PostgreSQL and BullMQ/Redis.
3. Auth.js with magic-link and GitHub OAuth.
4. Private workspace projects only.
5. Markdown as the only required MVP export.
6. OpenAI provider adapter with configurable model keys.
7. Proposed retention defaults in section 10.

Billing, public projects, additional export formats, and production deployment provider remain outside approved MVP scope.