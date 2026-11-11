# ADR 0001: Technology stack

- Status: Proposed
- Date: 2026-11-11

## Context

The MVP is one workflow for one founder on Gmail. It needs an HTTP API, background processing, a relational store with strong isolation, a web app for approvals, and a model call. It will be built largely with coding agents, so common, well-documented tools reduce errors.

## Decision

| Concern | Choice |
| --- | --- |
| Backend | Python with FastAPI |
| Database | PostgreSQL |
| Frontend | Next.js with TypeScript |
| Local environment | Docker Compose |
| LLM | Accessed through an `LLMProvider` interface; no provider-specific types outside the Intelligence package |

## Alternatives considered

- TypeScript for the backend as well. One language across the codebase, but the Python ecosystem for model evaluation and data handling is broader.
- A document or key-value store. Rejected: the workflow needs transactions across state, events and jobs, plus row-level security.
- Calling one model vendor's SDK directly throughout. Rejected: the blueprint states the moat must not depend on a particular model, and tests need a fixture provider.

## Consequences

- Two languages and two toolchains to maintain.
- Typed API contracts between backend and frontend must be generated from the OpenAPI schema to stay in sync.
- The provider interface limits use of vendor-specific features until the interface is extended.
