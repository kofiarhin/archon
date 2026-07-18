# ADR 0002: MVP implementation stack

- **Status:** Accepted
- **Date:** 2026-07-18
- **Accepted by:** PR #2

## Context

The approved Archon PRD and architecture require a TypeScript web application, Node.js API, PostgreSQL persistence, a Redis-backed worker queue, structured AI outputs, and isolated workspace authorization. The implementation plan needs concrete technologies so work can begin in small, testable increments.

## Decision

Use the following MVP stack:

- Next.js, React, and TypeScript for the web application and API boundary
- a separately runnable TypeScript worker using shared packages
- PostgreSQL with Prisma ORM and migrations
- Redis with BullMQ for durable background-job coordination
- Zod for shared versioned schemas and runtime validation
- Auth.js with email magic-link and GitHub OAuth
- OpenAI Responses API behind a provider-neutral adapter
- Vitest for unit and integration tests
- Playwright for browser acceptance tests
- Docker Compose for local PostgreSQL and Redis
- GitHub Actions for required checks

Projects are private to a workspace in the MVP. Markdown is the only required export format. Object storage is optional and used only when export bundles exceed the configured inline threshold.

## Constraints

- Provider and model identifiers remain configuration rather than domain logic.
- Authentication and authorization are enforced server-side.
- AI output is untrusted until schema and consistency validation pass.
- Approval applies to immutable artifact revisions and is never inherited automatically.
- Coding-agent prompts may be produced only for explicitly approved implementation tasks and never authorize or execute implementation.
- Hosting-provider selection is deferred.

## Consequences

### Positive

- one language and shared schema system across web, API, worker, and tests
- explicit transaction and migration support
- independently scalable generation workloads
- provider-neutral domain boundaries
- deterministic local and CI test environments

### Negative

- worker and web releases remain coupled to a shared source version
- Redis and PostgreSQL are both required operational dependencies
- Auth.js and Prisma upgrades require deliberate compatibility review
- repository package boundaries must be enforced to prevent modular-monolith erosion

## Revisit when

- measured traffic or team ownership requires service extraction
- authentication requirements exceed the selected adapter
- provider diversification requires capabilities not represented by the adapter
- scale or cost data shows BullMQ or the selected deployment shape is unsuitable
