# ADR 0001: Modular monolith with a background worker

- **Status:** Proposed
- **Date:** 2026-07-18

## Context

Archon needs transactional project and artifact management alongside long-running, retryable AI generation. The MVP should remain simple to develop and operate while allowing generation workloads to scale independently.

## Decision

Start with a modular monolith for the web application and API, plus a separate background worker process using the same application modules and database.

Use a durable queue between the API and worker. Keep AI-provider integration behind an adapter.

## Consequences

### Positive

- simpler deployment and local development than microservices
- clear transactional boundaries for project and artifact state
- generation can scale independently from interactive traffic
- provider and model strategy can change behind the adapter

### Negative

- module boundaries require discipline inside one codebase
- worker and API releases remain coupled initially
- future service extraction may require refactoring

## Revisit when

- measured workload requires independent service ownership
- security isolation demands separate trust boundaries
- deployment coupling becomes a material constraint
