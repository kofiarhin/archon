# Archon MVP Implementation Plan

**Status:** Proposed for approval  
**Last updated:** 2026-07-18

## 1. Delivery principles

- Implement only approved MVP scope.
- Deliver vertically usable increments rather than isolated infrastructure layers.
- Keep every change reviewable, reversible, and covered by tests.
- Do not mark a milestone complete until its exit checks pass.
- Use one branch and pull request per independently reviewable work item.
- Prefer schema-first and test-first development for domain behavior, API contracts, authorization, and generation validation.
- Treat migrations, authentication, authorization, AI-provider integration, and export rendering as high-risk changes requiring explicit review.

## 2. Workstream order

1. Repository and CI foundation
2. Authentication and workspace isolation
3. Project intake and persistence
4. Artifact schemas and revision model
5. Queue and worker platform
6. Planning and clarification
7. Full artifact generation
8. Review, editing, and approval
9. Consistency validation
10. Export
11. Hardening and release verification

## 3. Phase 1 — Foundation

### 1.1 Workspace bootstrap

Deliverables:

- pnpm workspace
- `apps/web` and `apps/worker`
- shared TypeScript configuration
- ESLint, Prettier, and import-boundary rules
- environment validation package
- Docker Compose for PostgreSQL and Redis
- documented local-development commands

Acceptance checks:

- clean install succeeds from a fresh clone
- web and worker start locally
- invalid configuration fails with a safe error
- no application package imports from another application's internal modules

### 1.2 CI baseline

Deliverables:

- GitHub Actions workflow
- formatting check
- lint
- type-check
- unit/integration tests
- migration validation
- production build
- dependency and secret scanning where repository settings support it

Acceptance checks:

- workflow runs on pull requests
- failure in any required check blocks completion
- build artifacts never contain `.env` files or credentials

### 1.3 Database foundation

Deliverables:

- Prisma schema
- initial migration
- test database setup
- repository interfaces
- transactional helper
- outbox table and dispatcher contract

Acceptance checks:

- migrations apply to an empty database
- rollback/recovery procedure is documented
- integration tests isolate data between test cases

## 4. Phase 2 — Authentication and project intake

### 2.1 Authentication

Deliverables:

- Auth.js integration
- GitHub OAuth
- email magic-link flow
- secure session configuration
- login/logout pages
- authentication middleware

Acceptance checks:

- unauthenticated API requests return 401
- session cookies use secure production settings
- OAuth or email tokens never appear in logs
- authentication-provider failure produces a recoverable user-facing error

### 2.2 Workspace authorization

Deliverables:

- workspace and membership model
- authorization policy service
- request-context workspace resolution
- cross-tenant not-found behavior
- authorization audit events

Acceptance checks:

- a user cannot read, mutate, generate, approve, export, or delete another workspace's project
- authorization is tested at service and HTTP boundaries
- browser-supplied workspace identifiers cannot bypass membership checks

### 2.3 Project creation and intake

Deliverables:

- project list and create UI
- validated intake form
- immutable project-input versions
- project detail shell
- rename and archive commands

Acceptance checks:

- authenticated users can create and retrieve a project
- invalid input is rejected consistently in UI and API
- editing intake creates a new version
- archive hides the project from default lists without deleting history

**Phase exit:** An authenticated user can create and retrieve an isolated project.

## 5. Phase 3 — Artifact and job foundations

### 3.1 Versioned schemas

Deliverables:

- registry of artifact types
- Zod schemas for every MVP artifact
- schema version metadata
- fixtures for valid and invalid outputs
- Markdown renderer contracts

Acceptance checks:

- each artifact type has a parseable schema
- unknown schema versions fail safely
- schema changes include migration or backward-compatibility guidance

### 3.2 Artifact revisions

Deliverables:

- artifact and immutable revision persistence
- revision history API
- manual revision command
- approval/supersession state transitions
- optimistic concurrency

Acceptance checks:

- approved revisions cannot be mutated
- regeneration/manual edit never overwrites unrelated revisions
- stale edits return a conflict
- approval records actor and time

### 3.3 Queue and worker

Deliverables:

- BullMQ connection and queue contracts
- outbox dispatcher
- idempotent job handlers
- retries and dead-letter visibility
- progress model
- graceful shutdown

Acceptance checks:

- duplicate commands do not create duplicate effective work
- transient failures retry according to policy
- non-retryable failures remain visible
- workers recover queued jobs after restart

## 6. Phase 4 — Planning and clarification

### 4.1 Provider adapter

Deliverables:

- provider-neutral AI interface
- OpenAI Responses API implementation
- configurable model keys
- structured output parsing
- timeout, retry, and usage metadata
- fake provider for tests

Acceptance checks:

- provider SDK types do not escape the adapter
- malformed output cannot be persisted as valid content
- secrets and raw credentials are absent from logs
- tests run without external provider access

### 4.2 Planning stage

Deliverables:

- planning prompt/template
- planning schema
- material-gap detection
- generated assumptions and questions
- deterministic validation

Acceptance checks:

- incomplete input yields clarification questions
- complete input can become ready for generation
- questions and assumptions are persisted separately
- users must explicitly accept assumptions or answer material questions

### 4.3 Clarification UI

Deliverables:

- question list
- answer form
- assumption accept/reject controls
- readiness status

Acceptance checks:

- progress is keyboard accessible
- resolved and unresolved items are clearly distinguishable
- generation is blocked while material questions remain unresolved

## 7. Phase 5 — Package generation

### 5.1 Artifact generators

Implement generators in dependency order:

1. product summary, goals, non-goals, assumptions, questions
2. functional and non-functional requirements
3. technology stack and ADR candidates
4. system architecture
5. data model and Mermaid ER diagram
6. API specification
7. authentication, authorization, security, privacy, reliability, and scalability
8. repository structure
9. implementation roadmap
10. coding-agent prompts

Each generator requires:

- versioned prompt and schema
- valid and invalid fixtures
- deterministic validation rules
- persistence integration
- progress reporting
- unit and worker integration tests

### 5.2 Parent orchestration

Deliverables:

- parent generation job
- dependency graph for child artifact jobs
- shared normalized project context
- cancellation handling
- final package completeness check

Acceptance checks:

- partial failure does not mark the package complete
- retry resumes safely without duplicating valid revisions
- completed jobs reference exact input, prompt, schema, provider, and model versions

**Phase exit:** A project can produce a complete schema-valid draft package.

## 8. Phase 6 — Review and consistency

### 6.1 Artifact review UI

Deliverables:

- artifact navigation
- structured and Markdown views
- revision history
- manual edit flow
- approve/reject controls
- scoped regeneration

Acceptance checks:

- the user can edit or regenerate one artifact independently
- approved content is visually distinct from generated proposals
- action confirmations identify the affected artifact and revision

### 6.2 Consistency engine

Deliverables:

- normalized symbol table for entities, components, roles, technologies, and endpoints
- validation rule registry
- structured findings
- severity and blocking policy
- project-level validation summary

Initial blocking rules:

- missing required artifact
- contradictory approved decision
- undefined core entity/component/role
- duplicate API method/path
- invalid artifact schema
- invalid required Mermaid diagram

Acceptance checks:

- conflicts are surfaced rather than silently rewritten
- findings reference affected artifacts and paths
- blocking findings prevent project approval/export-as-approved

### 6.3 Project approval

Deliverables:

- completeness policy
- project approval command
- approved manifest
- audit event

Acceptance checks:

- project approval requires all required artifacts approved
- unresolved blocking findings prevent approval
- approval is reproducible from the stored manifest

**Phase exit:** Users can iteratively review and approve a coherent package.

## 9. Phase 7 — Export and hardening

### 7.1 Markdown export

Deliverables:

- deterministic document ordering
- export manifest
- Markdown renderer
- Mermaid validation
- inline or object-storage delivery
- download and deletion endpoints

Acceptance checks:

- export includes generation date, approval state, assumptions, and unresolved questions
- Mermaid diagrams remain text based and valid
- identical manifests produce identical content digests
- untrusted content cannot execute active HTML/script in previews

### 7.2 Reliability and operations

Deliverables:

- structured logging
- metrics and tracing
- queue and provider dashboards
- alerts
- rate limits and cost controls
- retention jobs
- backup/restore documentation
- incident runbook

Acceptance checks:

- operators can trace a request through parent and child jobs
- stuck jobs are detectable and recoverable
- provider outage degrades safely
- deletion and retention behavior is tested

### 7.3 Accessibility, performance, and security

Deliverables:

- keyboard and screen-reader review
- automated accessibility checks
- API and generation performance benchmarks
- authorization test suite
- dependency and configuration review
- threat-model verification

Acceptance checks:

- generation request is acknowledged within one second under target local/staging conditions
- progress is exposed by stage/artifact
- core flows meet WCAG 2.2 AA target
- no critical/high unresolved security findings

**Phase exit:** All MVP acceptance criteria in `docs/PRD.md` pass.

## 10. Suggested pull-request sequence

1. `build: initialize pnpm workspace and CI`
2. `feat: add database and configuration foundation`
3. `feat: add authentication and workspace authorization`
4. `feat: add project intake and immutable inputs`
5. `feat: add artifact schemas and revision model`
6. `feat: add BullMQ worker and outbox dispatch`
7. `feat: add AI provider adapter and planning stage`
8. `feat: add clarification workflow`
9. `feat: generate requirements and product artifacts`
10. `feat: generate architecture, data, and API artifacts`
11. `feat: generate security, roadmap, ADR, and agent prompts`
12. `feat: add artifact review and scoped regeneration`
13. `feat: add consistency validation and project approval`
14. `feat: add Markdown export`
15. `chore: harden observability, accessibility, and release checks`

Each PR should include its own acceptance evidence and avoid unrelated refactoring.

## 11. Release gate

The MVP may be released only when:

- all PRD acceptance criteria pass
- required CI checks are green
- migrations have been tested from an empty and previous supported schema
- authorization and tenant-isolation tests pass
- background-job recovery tests pass
- export fixtures validate
- accessibility review is complete
- threat model and operational runbook are reviewed
- retention and deletion defaults are explicitly approved
- a rollback plan exists for application and schema changes

## 12. Deferred work

The following must not be pulled into MVP implementation without a PRD revision:

- application source-code generation
- autonomous deployment
- infrastructure provisioning
- public project sharing
- real-time collaboration
- billing and organization administration
- visual drag-and-drop architecture editing
- OpenAPI, PDF, or infrastructure-as-code export beyond experiments not exposed as supported product behavior