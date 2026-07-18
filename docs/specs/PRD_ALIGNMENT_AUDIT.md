# Archon MVP Specification Alignment Audit

**Status:** Approved with PR #2  
**Audit date:** 2026-07-18  
**Compared against:** `docs/PRD.md`, `docs/ARCHITECTURE.md`, `docs/SECURITY.md`, `docs/ROADMAP.md`, and ADR 0001

## Conclusion

The MVP implementation specification and plan align with the approved product requirements. They preserve the MVP boundary, cover every required product artifact and workflow, maintain explicit human approval, and include the required authorization, reliability, accessibility, observability, export, and testing controls.

No application implementation is authorized by this audit. Implementation work must follow the approved plan in independently reviewable pull requests and pass the acceptance gate for each increment.

## Requirement traceability

| PRD requirement | Specification and plan coverage | Result |
|---|---|---|
| Project creation, rename, retrieval, and archive | Project API, immutable inputs, project intake phase | Aligned |
| Clarification before final generation | Planning job, material-gap detection, questions and assumptions workflow | Aligned |
| Complete architecture package | Versioned artifact registry and package-generation sequence | Aligned |
| Schema-constrained generation | Zod schemas, provider adapter, structured output validation | Aligned |
| Cross-artifact consistency | Symbol table, deterministic rules, blocking findings | Aligned |
| Independent editing and regeneration | Immutable revisions, scoped regeneration, optimistic concurrency | Aligned |
| Explicit approval states | Revision approval, decision records, approved manifests, audit events | Aligned |
| Markdown export with Mermaid source | Deterministic Markdown renderer, Mermaid validation, export manifest | Aligned |
| Workspace authorization | Server-side membership checks, tenant-scoped repositories, denial tests | Aligned |
| Retryable and recoverable generation | Outbox, BullMQ jobs, idempotency, retry classification, stale-job recovery | Aligned |
| Traceability and observability | Request/job IDs, provider/model/schema metadata, metrics and alerts | Aligned |
| Keyboard access and WCAG 2.2 AA target | UX requirements, Playwright coverage, accessibility release evidence | Aligned |
| Automated core-workflow tests | Unit, integration, contract, browser, security, recovery, and evaluation suites | Aligned |

## Scope audit

The specification does not add the following excluded MVP capabilities:

- complete application source-code generation
- autonomous deployment
- database migration execution on behalf of generated projects
- automatic infrastructure provisioning
- public project sharing
- real-time collaboration
- billing or enterprise administration
- visual drag-and-drop architecture editing

Restore endpoints, object-storage support for large export bundles, and operational deployment guidance are implementation support capabilities rather than new product scope.

## Corrections and binding clarifications

### Approval bookkeeping

ADR 0001 is marked `Accepted` because PR #1 approved the modular-monolith and worker direction. PR #2 records the accepted implementation-stack choices in ADR 0002.

### Coding-agent prompts

The PRD permits coding-agent prompts only for approved implementation tasks. Therefore:

- prompts must not be generated for a merely proposed or unresolved task;
- each prompt must reference an explicitly approved task, its acceptance criteria, relevant artifact revisions, and scope exclusions;
- prompt generation does not execute code, modify repositories, deploy software, run migrations, or authorize implementation;
- regenerated prompts create new immutable artifact revisions and do not inherit approval automatically.

This clarification is normative and resolves any ambiguity in the package-generation sequence.

### Export state

Exports may include proposed material only when it is clearly labelled as proposed and the export manifest records that selection. An export must not be represented as an approved package unless all required artifacts are approved and no blocking validation finding remains unresolved.

### Retention

The retention durations in the implementation specification become the initial MVP defaults when PR #2 is merged. They remain configurable operational policy and must be reviewed before production release.

## Remaining non-blocking implementation decisions

The following can be selected during implementation without changing product scope, provided they are documented and reviewed:

- exact OpenAI model configuration and fallback order
- hosting and managed-service vendors
- object-storage inline-size threshold
- numeric rate, payload, queue-concurrency, alert, and cost limits
- final synthetic quality benchmark thresholds

Any change to the MVP product boundary or required outputs requires a PRD revision.
