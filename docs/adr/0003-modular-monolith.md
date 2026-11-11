# ADR 0003: Modular monolith with enforced layer boundaries

- Status: Proposed
- Date: 2026-11-11

## Context

The architecture has ten layers with defined responsibilities. The blueprint asks for clear boundaries and warns against coupling every layer to every other. The MVP has one founder and one workflow; operating ten services would add deployment and consistency work without a matching benefit.

## Decision

- One backend codebase and one deployable image, run as two process types: `api` and `worker`.
- Each layer is a Python package with a public interface module. Other packages import only that module.
- Allowed imports follow the call rules in [`../architecture.md`](../architecture.md#3-call-rules) and are checked in CI by an import-boundary tool.
- Each database table has one owning layer that writes it.
- Channel client libraries are imported only by Integration. The LLM client is imported only by Intelligence.

## Alternatives considered

- One service per layer. Rejected for the MVP: distributed transactions, network failure modes and deployment overhead for no current scaling need.
- A single package without enforced boundaries. Rejected: boundaries erode quickly, especially with generated code.

## Consequences

- Layers can share a database transaction, which the workflow relies on for writing state, events and jobs together.
- A layer can later be extracted into a service along its interface if load or isolation requires it.
- The boundary check must be maintained as layers are added; a failing check blocks merges.
