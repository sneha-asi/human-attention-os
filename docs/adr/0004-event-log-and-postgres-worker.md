# ADR 0004: Append-only event log and PostgreSQL-backed worker

- Status: Proposed
- Date: 2026-11-11

## Context

The system must show why each action was taken, must not lose work when a process dies, and must run timers (reminders, expiries, outcome checks). The MVP volume is one mailbox.

## Decision

- An append-only `events` table records every observation, proposal, decision, command and outcome with actor, trace ID and references to the records involved. Events carry no message content or credentials.
- Current state is kept in ordinary tables (`workflows`, `actions`, and others) and updated in the same transaction as the corresponding event. Replaying a Workflow's events must reproduce its state; a test checks this.
- Background work and timers use a `jobs` table in PostgreSQL. Workers claim jobs with `FOR UPDATE SKIP LOCKED` and a lease. The next job is inserted in the same transaction as the state change that requires it.
- Job handlers are idempotent and begin by checking the Workflow is in the expected state.

## Alternatives considered

- Full event sourcing with state derived only from events. Rejected for the MVP: more machinery than one workflow needs. The replay test keeps the option open.
- A durable workflow engine such as Temporal. Deferred: a strong fit once there are many workflow types, but it adds an operational dependency and a programming model before the first workflow is proven.
- A message broker with separate task queue. Rejected for the MVP: enqueueing would no longer be transactional with state changes without an outbox, which is what the `jobs` table already is.

## Consequences

- No extra infrastructure beyond PostgreSQL.
- Throughput is bounded by polling the `jobs` table; acceptable at pilot scale and revisited before multi-founder load.
- Because events hold references only, deleting or expiring content does not require rewriting the log.
- Moving to a workflow engine later means porting the transition function and timers; the state machine documents remain valid.
