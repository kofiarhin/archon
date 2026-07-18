# Archon API and Data Specification

**Status:** Proposed for approval  
**Last updated:** 2026-07-18

## 1. Conventions

- Base path: `/api/v1`
- Media type: `application/json`
- Authentication: secure, HTTP-only session cookie
- Identifiers: opaque UUIDs
- Timestamps: UTC ISO 8601
- Pagination: cursor-based
- Mutating generation/export commands: `Idempotency-Key` required
- Optimistic concurrency: revision/version field required for edits

## 2. Resource endpoints

### Workspaces

- `GET /workspaces` — list memberships
- `GET /workspaces/{workspaceId}` — retrieve workspace
- `GET /workspaces/{workspaceId}/members` — list members

### Projects

- `POST /workspaces/{workspaceId}/projects`
- `GET /workspaces/{workspaceId}/projects`
- `GET /projects/{projectId}`
- `PATCH /projects/{projectId}`
- `POST /projects/{projectId}/archive`
- `POST /projects/{projectId}/restore`

Project creation body:

```json
{
  "name": "Architecture Studio",
  "input": {
    "description": "...",
    "targetUsers": ["..."],
    "coreWorkflows": ["..."],
    "constraints": ["..."],
    "expectedScale": "...",
    "preferredTechnologies": ["..."],
    "prohibitedTechnologies": ["..."]
  }
}
```

### Inputs and clarification

- `GET /projects/{projectId}/inputs`
- `POST /projects/{projectId}/inputs` — create immutable input version
- `POST /projects/{projectId}/analysis-jobs`
- `GET /projects/{projectId}/questions`
- `POST /projects/{projectId}/questions/{questionId}/answer`
- `POST /projects/{projectId}/assumptions/{assumptionId}/accept`
- `POST /projects/{projectId}/assumptions/{assumptionId}/reject`

### Generation

- `POST /projects/{projectId}/generation-jobs`
- `GET /projects/{projectId}/generation-jobs`
- `GET /generation-jobs/{jobId}`
- `POST /generation-jobs/{jobId}/retry`
- `POST /generation-jobs/{jobId}/cancel`

Generation body:

```json
{
  "inputVersion": 3,
  "artifactTypes": ["requirements", "system_architecture"],
  "mode": "full"
}
```

`mode` is `full` or `section_regeneration`. Full generation requires all material questions resolved or assumptions accepted.

### Artifacts

- `GET /projects/{projectId}/artifacts`
- `GET /artifacts/{artifactId}`
- `GET /artifacts/{artifactId}/revisions`
- `GET /artifact-revisions/{revisionId}`
- `POST /artifacts/{artifactId}/revisions` — manual edit
- `POST /artifacts/{artifactId}/regeneration-jobs`
- `POST /artifact-revisions/{revisionId}/approve`
- `POST /artifact-revisions/{revisionId}/reject`

Manual edit body:

```json
{
  "baseRevisionId": "uuid",
  "content": {},
  "changeSummary": "Clarified tenancy boundaries"
}
```

A stale `baseRevisionId` returns `409 REVISION_CONFLICT`.

### Decisions, assumptions, and questions

- `GET /projects/{projectId}/decisions`
- `POST /projects/{projectId}/decisions`
- `PATCH /decisions/{decisionId}`
- `GET /projects/{projectId}/assumptions`
- `GET /projects/{projectId}/questions`

### Validation findings

- `GET /projects/{projectId}/validation-findings`
- `GET /artifact-revisions/{revisionId}/validation-findings`

### Exports

- `POST /projects/{projectId}/exports`
- `GET /projects/{projectId}/exports`
- `GET /exports/{exportId}`
- `GET /exports/{exportId}/download`
- `DELETE /exports/{exportId}`

Export body:

```json
{
  "format": "markdown",
  "revisionSelection": "approved_with_latest_proposals",
  "includeUnresolvedQuestions": true
}
```

## 3. Error codes

- `AUTHENTICATION_REQUIRED` — 401
- `ACCESS_DENIED` — 403
- `RESOURCE_NOT_FOUND` — 404, including inaccessible cross-workspace resources
- `VALIDATION_FAILED` — 422
- `INVALID_PROJECT_STATE` — 409
- `REVISION_CONFLICT` — 409
- `IDEMPOTENCY_CONFLICT` — 409
- `MATERIAL_QUESTIONS_UNRESOLVED` — 409
- `JOB_NOT_RETRYABLE` — 409
- `RATE_LIMITED` — 429
- `PROVIDER_UNAVAILABLE` — 503
- `INTERNAL_ERROR` — 500

Cross-workspace resource access returns the same not-found response as a missing resource.

## 4. Relational data model

### users

- `id` UUID primary key
- `display_name`
- `email` nullable, normalized where present
- `created_at`, `updated_at`

### auth_accounts / auth_sessions

Provider-specific authentication records managed by the authentication adapter. No provider token may be exposed to application clients or logs.

### workspaces

- `id`
- `name`
- `created_by_user_id`
- `created_at`, `updated_at`, `archived_at`

### workspace_memberships

- `workspace_id`
- `user_id`
- `role` (`owner`, `member`)
- `created_at`

Unique: `(workspace_id, user_id)`.

### projects

- `id`
- `workspace_id`
- `name`
- `status`
- `current_input_version`
- `version` for optimistic concurrency
- `created_by_user_id`
- `created_at`, `updated_at`, `archived_at`, `deleted_at`

Indexes: `(workspace_id, updated_at)`, `(workspace_id, status)`.

### project_inputs

- `id`
- `workspace_id`
- `project_id`
- `version`
- structured JSON fields for intake data
- `content_digest`
- `created_by_user_id`
- `created_at`

Unique: `(project_id, version)`.

### artifacts

- `id`
- `workspace_id`
- `project_id`
- `type`
- `current_revision_number`
- `created_at`

Unique: `(project_id, type)`.

### artifact_revisions

- `id`
- `workspace_id`
- `artifact_id`
- `revision_number`
- `schema_version`
- `status`
- `content_json`
- `content_markdown` nullable derived representation
- `source` (`generated`, `manual`)
- `based_on_revision_id` nullable
- `project_input_id`
- `generation_job_id` nullable
- `created_by_user_id` nullable for worker-generated content
- `approved_by_user_id` nullable
- `created_at`, `approved_at`, `superseded_at`

Unique: `(artifact_id, revision_number)`.

### decisions

- `id`, `workspace_id`, `project_id`
- `artifact_revision_id` nullable
- `title`, `statement`, `rationale`
- `status` (`proposed`, `approved`, `rejected`, `superseded`, `unresolved`)
- `source` (`user`, `generated`)
- `created_by_user_id` nullable
- `resolved_by_user_id` nullable
- timestamps

### assumptions

- `id`, `workspace_id`, `project_id`
- `artifact_revision_id` nullable
- `statement`, `impact`, `status`
- `source`
- acceptance/rejection actor and timestamps

### open_questions

- `id`, `workspace_id`, `project_id`
- `question`, `reason`, `materiality`
- `status` (`open`, `answered`, `accepted_assumption`, `dismissed`)
- `answer` nullable
- actor and timestamps

### generation_jobs

- `id`, `workspace_id`, `project_id`
- `parent_job_id` nullable
- `type`, `state`, `attempt`
- `idempotency_key`
- `input_version`, `schema_version`, `prompt_version`
- `provider`, `model_key`
- `requested_artifact_types`
- `progress_json`
- `failure_code`, `failure_message_safe`
- `queued_at`, `started_at`, `completed_at`, `cancelled_at`

Unique: `(workspace_id, idempotency_key)` for active command scope.

### generation_usage

- `generation_job_id`
- `provider_request_id` nullable
- `latency_ms`
- input/output token counts nullable
- `estimated_cost_minor_units` nullable
- `created_at`

No prompt or response body is stored here.

### validation_findings

- `id`, `workspace_id`, `project_id`
- `artifact_revision_id` nullable
- `generation_job_id` nullable
- `rule_code`, `severity`, `message`, `path`, `details_json`
- `created_at`, `resolved_at`

### exports

- `id`, `workspace_id`, `project_id`
- `format`, `state`
- `manifest_json`
- `content_digest`
- `storage_key` nullable
- `size_bytes` nullable
- `created_by_user_id`
- `created_at`, `completed_at`, `deleted_at`

### audit_events

- `id`, `workspace_id`
- `project_id` nullable
- `actor_user_id` nullable
- `actor_type` (`user`, `worker`, `system`)
- `event_type`
- `target_type`, `target_id`
- `metadata_json` with allowlisted non-sensitive fields
- `request_id`, `created_at`

### outbox_events

- `id`
- `topic`
- `aggregate_type`, `aggregate_id`
- `payload_json`
- `available_at`, `published_at`, `attempts`

## 5. Database invariants

- Every project-owned row must match the project's workspace.
- Only one approved revision may exist per artifact.
- Revision numbers increase monotonically.
- Approved revisions are immutable.
- A project cannot enter `review` without the required artifact set.
- A project cannot enter `approved` while required artifacts lack approved revisions or material findings remain unresolved.
- Jobs reference an existing immutable input version.
- Exports reference an immutable revision manifest.

These rules should be enforced using a combination of database constraints, transactions, and domain validation.

## 6. Authorization matrix

| Action | Owner | Member |
|---|---:|---:|
| View workspace projects | Yes | Yes |
| Create and edit project | Yes | Yes |
| Start generation/export | Yes | Yes |
| Approve artifact revision | Yes | Yes |
| Archive project | Yes | Yes |
| Manage workspace membership | Yes | No |
| Archive workspace | Yes | No |

All checks are evaluated server-side from authenticated membership records.