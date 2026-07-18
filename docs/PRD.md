# Archon Product Requirements Document

**Status:** Draft for approval  
**Last updated:** 2026-07-18  
**Product:** Archon  
**Tagline:** From idea to architecture.

## 1. Product summary

Archon is an AI-powered software architecture studio that transforms a plain-language product idea into a structured, reviewable, implementation-ready engineering blueprint.

The first release focuses on architecture documentation and delivery planning. It does not generate or deploy a complete production application.

## 2. Problem

Developers, freelancers, founders, students, and small teams often begin implementation before requirements, architecture, data design, APIs, security boundaries, and delivery phases are clear. This creates avoidable rework, inconsistent technical decisions, and weak handoffs to humans or coding agents.

## 3. Goal

Enable a user to describe an application idea and receive a coherent architecture package that can be reviewed, edited, exported, and used to begin implementation.

## 4. Primary user

The initial primary user is a solo developer or small technical team that needs to turn an early-stage application idea into an actionable engineering plan.

Secondary users include freelancers, startup founders, students, technical product managers, and engineering consultants.

## 5. MVP scope

### 5.1 Idea intake

Users can provide:

- product description
- target users
- core workflows
- constraints and preferences
- expected scale
- preferred or prohibited technologies

Archon must identify material gaps and request clarification before producing a final blueprint.

### 5.2 Generated architecture package

The MVP generates:

1. product summary and problem statement
2. goals, non-goals, assumptions, and open questions
3. functional and non-functional requirements
4. recommended technology stack with rationale and alternatives
5. system context and component architecture
6. core data model and Mermaid ER diagram
7. API resource and endpoint specification
8. authentication and authorization flow
9. security, privacy, reliability, and scalability considerations
10. repository and folder structure recommendation
11. phased implementation roadmap
12. architecture decision records
13. coding-agent prompts for approved implementation tasks

### 5.3 Review and revision

Users can:

- review generated sections independently
- regenerate a selected section without replacing the entire blueprint
- edit content manually
- mark decisions as approved, proposed, or unresolved
- maintain assumptions and open questions

### 5.4 Export

Users can export the architecture package as Markdown. Mermaid diagrams must remain valid text-based diagrams in the export.

## 6. Out of scope for MVP

- autonomous production deployment
- complete application code generation
- direct database migration execution
- automatic infrastructure provisioning
- visual drag-and-drop architecture editing
- real-time multiplayer collaboration
- billing and enterprise administration
- guarantees that generated architecture is correct without human review

## 7. Core workflow

1. User creates a project.
2. User submits an application idea and known constraints.
3. Archon analyzes the request and identifies missing decisions.
4. User answers clarification questions or accepts documented assumptions.
5. Archon generates the architecture package using structured outputs.
6. Archon validates completeness and cross-document consistency.
7. User reviews, edits, regenerates, and approves sections.
8. User exports the approved blueprint.

## 8. Functional requirements

### FR-1 Project management

- Users can create, rename, open, and archive architecture projects.
- Each project stores its inputs, generated artifacts, revisions, decisions, assumptions, and open questions.

### FR-2 Structured generation

- Generated sections must conform to versioned schemas.
- Failures must not leave a project in a partially approved state.
- The system must distinguish generated content from user-approved decisions.

### FR-3 Consistency validation

- Named components, entities, roles, and technologies must remain consistent across artifacts.
- Conflicts must be surfaced instead of silently resolved.

### FR-4 Section revision

- A user can regenerate or edit one artifact without overwriting unrelated approved artifacts.
- Regeneration must preserve relevant approved constraints.

### FR-5 Export

- Users can export a complete Markdown package.
- Exported documents must include generation date, assumptions, unresolved questions, and approval state.

## 9. Non-functional requirements

### NFR-1 Security

- Secrets must never be included in prompts, logs, exports, or source control.
- Project access must be authorized server-side.
- Inputs and outputs must be treated as untrusted content.

### NFR-2 Reliability

- Generation jobs must be retryable and idempotent where practical.
- Partial failures must be visible and recoverable.

### NFR-3 Performance

- The interface should acknowledge a generation request within one second.
- Long-running generation must expose progress by artifact or stage.

### NFR-4 Observability

- Generation requests must have traceable job IDs.
- The system must record provider, model, schema version, latency, token usage, and validation outcome without recording secrets.

### NFR-5 Accessibility

- Core workflows must be keyboard accessible.
- The application should target WCAG 2.2 AA for production release.

## 10. Proposed architecture

The initial implementation is expected to use:

- React and TypeScript frontend
- Node.js API
- PostgreSQL with an ORM
- Redis-backed job queue
- OpenAI Responses API with structured outputs
- object storage for generated export bundles if required
- Docker for local development
- GitHub Actions for automated checks

These choices remain proposed until the architecture decision records are approved.

## 11. Data concepts

Core entities:

- User
- Workspace
- Project
- ProjectInput
- Artifact
- ArtifactRevision
- Decision
- Assumption
- OpenQuestion
- GenerationJob
- Export

## 12. AI generation requirements

- Use schema-constrained outputs for every artifact.
- Separate planning, generation, validation, and synthesis stages.
- Treat model output as untrusted until schema and consistency validation pass.
- Preserve source inputs and generation metadata for reproducibility.
- Never label an AI-generated decision as approved without explicit user action.

## 13. Success metrics

Initial product metrics:

- percentage of projects producing a complete validated package
- time from idea submission to first reviewable package
- artifact regeneration rate
- export completion rate
- percentage of generated packages approved without major structural rewrite
- user-reported usefulness for beginning implementation

## 14. Acceptance criteria

The MVP is acceptable when:

- a user can create a project and submit an idea with constraints
- the system identifies missing material inputs
- the system generates all required MVP artifacts using validated structured outputs
- generated entities and technology decisions are cross-checked for consistency
- a user can edit or regenerate one artifact independently
- approved decisions remain distinct from proposals and assumptions
- the complete project can be exported as readable Markdown with valid Mermaid diagrams
- unauthorized users cannot access another user's projects
- generation failures can be retried without corrupting approved artifacts
- automated tests cover the core project, generation, validation, authorization, and export workflows

## 15. Risks

- hallucinated or internally inconsistent architecture advice
- provider cost and latency growth with large projects
- prompt injection through project inputs or imported content
- users treating generated plans as guaranteed production guidance
- over-scoping the MVP into code generation or deployment automation

## 16. Open decisions requiring approval

- final frontend and backend framework choices
- authentication provider or in-house authentication
- exact AI model and fallback strategy
- retention and deletion policy
- public versus private projects
- usage limits and billing model
- supported export formats beyond Markdown
- quality benchmark and evaluation dataset
