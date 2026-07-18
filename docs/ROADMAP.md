# Archon Development Roadmap

**Status:** Proposed  
**Last updated:** 2026-07-18

## Phase 0 — Product definition

- approve the PRD and MVP boundaries
- approve the primary user and required outputs
- approve architecture, security baseline, and evaluation approach
- define repository conventions and CI checks

**Exit condition:** Product scope and acceptance criteria are approved.

## Phase 1 — Foundation

- establish application workspace and local development environment
- add authentication and workspace boundaries
- implement project creation and idea intake
- define versioned artifact schemas
- establish database migrations, testing, linting, and CI

**Exit condition:** An authenticated user can create and retrieve an isolated project.

## Phase 2 — Generation pipeline

- implement queued generation jobs
- integrate the AI provider adapter
- add planning and clarification stages
- generate the first structured artifact set
- validate schemas and persist immutable revisions

**Exit condition:** A project can produce a complete schema-valid draft package.

## Phase 3 — Review and consistency

- add artifact review and editing
- support section-level regeneration
- track proposals, decisions, assumptions, and open questions separately
- implement cross-artifact consistency checks
- expose job progress and recoverable failures

**Exit condition:** Users can iteratively review and approve a coherent package.

## Phase 4 — Export and hardening

- add Markdown export with valid Mermaid diagrams
- add authorization and security tests
- add observability, rate limits, and cost controls
- complete accessibility and reliability checks
- run architecture-quality evaluations against a curated test set

**Exit condition:** The MVP acceptance criteria in `PRD.md` pass.

## Post-MVP candidates

- visual architecture editor
- collaborative review
- OpenAPI and infrastructure exports
- GitHub repository bootstrapping
- starter-code generation
- organization controls and billing

Post-MVP candidates are not approved scope until added to an accepted PRD revision.
