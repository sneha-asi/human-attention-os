# Axon — System Architecture (HLD)

This document defines the ten layers, the three cross-cutting capabilities, the rules for which layer may call which, and the stack. It covers the whole system and marks what the MVP builds.

Related documents:

- Requirements, MVP scope and acceptance criteria: [`prd.md`](prd.md)
- Entities and relations: [`domain-model.md`](domain-model.md)
- Rules that must always hold: [`invariants.md`](invariants.md)
- State machine and low-level design of the first workflow: [`workflows/investor-document-request.md`](workflows/investor-document-request.md)
- Channel API feasibility: [`feasibility.md`](feasibility.md)
- Test-case format and cases: [`evaluation.md`](evaluation.md)
- Decisions and their alternatives: [`adr/`](adr/)

## 1. Shape of the system

Axon is one deployable backend (a modular monolith, see [ADR 0003](adr/0003-modular-monolith.md)) with two process types and a web app:

| Process | Role |
| --- | --- |
| `api` | FastAPI app. Serves the web app, receives founder decisions, manages Permissions and channel connections. |
| `worker` | Runs ingestion polling, workflow steps, execution, outcome checks and timers from a PostgreSQL-backed job queue. |
| `web` | Next.js app. Founder inbox, approval screen, workflow timeline, settings. |

Each layer is a Python package under `axon/`. Layers communicate through typed interfaces and through events in an append-only log ([ADR 0004](adr/0004-event-log-and-postgres-worker.md)).

Four kinds of record are kept distinct throughout:

| Kind | Meaning | Produced by |
| --- | --- | --- |
| Observation | Something received or read from a channel | Integration, Outcome Tracking |
| Proposal | What the model and planner suggest doing | Intelligence, Decision & Orchestration |
| Command | A proposal that has been authorised | Policy gate or founder Approval |
| Event | A fact about what the system did | Every layer |

A proposal becomes a command only through the policy gate ([ADR 0002](adr/0002-deterministic-policy-gate.md)). Only commands reach a channel.

## 2. Data flow

```text
Channel API (Gmail)
   │ fetch_changes
   ▼
1 Integration ──raw item──▶ 2 Data Normalisation ──▶ 3 Data Storage
                                                        │ event: message.ingested
                                                        ▼
                                         7 Decision & Orchestration
                                            │   │   │        │
                           ┌────────────────┘   │   │        └──────────────┐
                           ▼                    ▼   ▼                       ▼
                 4 Identity & Relationships  5 Context & Memory   10 Founder Intelligence
                                                    │
                                                    ▼
                                            6 Intelligence ──▶ LLM provider
                                                    │ interpretation + draft
                                                    ▼
                                         7 builds proposal ──▶ Policy gate
                                                    │ auto-execute / approval required / blocked
                                                    ▼
                                (founder Approval via web app when required)
                                                    │ command
                                                    ▼
                                             8 Execution ──▶ 1 Integration ──▶ Channel API
                                                    │
                                                    ▼
                                          9 Outcome Tracking ──▶ 1 Integration (observe)
                                                    │ event: outcome.verified
                                                    ▼
                                         7 closes or escalates the workflow
```

## 3. Call rules

A layer may import and call only the layers listed for it. CI enforces this with an import-boundary check (see [INV-12](invariants.md)).

| Layer | May call | Called by |
| --- | --- | --- |
| 1 Integration | Channel APIs, 2 (hand-off of raw items), 3 (cursors, credentials) | 8, 9, worker scheduler |
| 2 Data Normalisation | 3 | 1 |
| 3 Data Storage | PostgreSQL, object store | All layers, through repositories |
| 4 Identity & Relationships | 3 | 7 |
| 5 Context & Memory | 3 | 7 |
| 6 Intelligence | LLM provider | 7 |
| 7 Decision & Orchestration | 3, 4, 5, 6, 8, 9, 10, policy gate | Worker (jobs), `api` (founder decisions) |
| 8 Execution | 1, 3, policy gate (authorisation check) | 7 |
| 9 Outcome Tracking | 1, 3 | 7 |
| 10 Founder Intelligence | 3 | 7 |

Rules that follow from the table:

- Only Integration talks to channel APIs. Only Intelligence talks to the LLM provider.
- Intelligence has no storage or channel access. It receives everything it needs as arguments and returns structured output.
- Layers 4, 5, 6, 9 and 10 do not call each other. Orchestration passes results between them.
- Each table has one owning layer that writes it. Other layers read through the owner's repository interface. Table ownership is listed in the [workflow LLD](workflows/investor-document-request.md#5-tables).
- The `api` process calls Orchestration for founder decisions and the governance module for Permission changes. It does not call Execution or Integration's `execute`.

## 4. The ten layers

### Layer 1: Integration Layer

| | |
| --- | --- |
| Responsibility | Connect channel accounts, ingest messages and updates, send on behalf of Execution, read external state for Outcome Tracking. |
| Inputs | OAuth grants, channel API responses, commands from Execution, observe requests from Outcome Tracking. |
| Outputs | Raw channel items with provider IDs and a cursor; execution results; observed external state. |
| Must not | Interpret content, decide whether to send, or hold business state other than cursors and credentials. |

Every channel implements one connector contract:

```text
authorize()                           obtain and refresh credentials
capabilities()                        what this channel can read and do
fetch_changes(cursor)                 raw items since cursor, plus new cursor
validate(command)                     reject malformed commands before any call
execute(command, idempotency_key)     perform the side effect
observe(external_ref)                 read current external state of an object
reconcile(command)                    determine whether a command with unknown outcome took effect
```

MVP builds: the Gmail connector for one mailbox, polling for changes. A fake connector with the same contract for tests and evaluation.
Deferred: WhatsApp, Slack, Teams, LinkedIn, calendar, push-based ingestion.

### Layer 2: Data Normalisation Layer

| | |
| --- | --- |
| Responsibility | Convert raw channel items into the internal Conversation and Message representation. |
| Inputs | Raw items from Integration. |
| Outputs | Normalised Message, Conversation and Attachment metadata with a reference back to the raw item. |
| Must not | Call channel APIs, call the LLM, drop fields silently. Unparseable items are recorded as failed normalisation with the raw reference kept. |

Normalisation is deterministic. The same raw item always yields the same normalised record.

MVP builds: Gmail message and thread mapping; plain-text body extraction (HTML stripped, quoted history separated); participant and header extraction, including sender-authentication results.
Deferred: mappings for other channels; rich content.

### Layer 3: Data Storage Layer

| | |
| --- | --- |
| Responsibility | Persist records, the event log and blobs. Enforce founder isolation, encryption of credentials, retention and deletion. |
| Inputs | Writes from owning layers through repositories. |
| Outputs | Reads through repositories; `message.ingested` and other storage events. |
| Must not | Contain business rules about what to do with a message. Serve a query that is not scoped to one founder. |

Message bodies and attachments are stored by reference. Events carry IDs and references, never content, so content can be deleted without rewriting the log.

MVP builds: PostgreSQL schema with row-level security on `founder_id`; local-disk object store behind an interface; field encryption for OAuth tokens; retention job for message bodies and raw items; founder data deletion.
Deferred: customer-hosted storage, managed key service, vector index.

### Layer 4: Identity & Relationships Layer

| | |
| --- | --- |
| Responsibility | Map a channel handle to a Contact and state the Contact's relationship to the Founder. |
| Inputs | Founder ID, channel, handle, sender-authentication result. |
| Outputs | Identity match: Contact or none, relationship category, whether the sender is verified, match basis. |
| Must not | Create or merge Contacts on model output alone. Report an unauthenticated sender as verified. |

MVP builds: founder-maintained Contacts with relationship category and email Identities; exact-match resolution on email address; verified flag derived from Gmail's authentication headers.
Deferred: cross-channel identity linking, suggested merges, inferred relationship categories.

### Layer 5: Context & Memory Layer

| | |
| --- | --- |
| Responsibility | Assemble the history relevant to one message. |
| Inputs | Founder ID, Message, resolved Contact. |
| Outputs | Context bundle: the current Conversation, recent Conversations with the Contact, open Tasks and Commitments involving the Contact, prior Actions toward the Contact (for example, whether a document was already sent), candidate Documents. Each item carries its source ID. |
| Must not | Return data from another founder. Return items without a source reference. Summarise with the LLM (it passes records; Intelligence reads them). |

MVP builds: structured queries over Storage, bounded by count and age.
Deferred: semantic retrieval, long-term summaries, Commitment extraction from history.

### Layer 6: Intelligence Layer

| | |
| --- | --- |
| Responsibility | Interpret a message and draft a reply. |
| Inputs | Normalised Message, context bundle, founder preferences, list of candidate Documents (ID and title only). |
| Outputs | Interpretation: intent, requested item, urgency, risks, confidence, chosen Document ID or none. Draft: reply body text. Both validated against a schema; model ID and prompt version recorded. |
| Must not | Decide whether an action is permitted. Choose recipients. Access storage or channels. Return free-form output that bypasses schema validation. |

Message content is treated as untrusted data. The model's output can influence the reply body and which registered Document is proposed. It cannot set the recipient, the autonomy tier or the authorisation.

MVP builds: one interpretation prompt and one drafting prompt behind an `LLMProvider` interface; a fixture provider for tests.
Deferred: commitment and decision extraction, multi-intent messages, model routing.

### Layer 7: Decision & Orchestration Layer

| | |
| --- | --- |
| Responsibility | Own each Workflow. Run its steps, build the proposal, obtain a policy decision, request founder approval when required, hand commands to Execution, close or escalate. |
| Inputs | `message.ingested` events, founder decisions from the `api`, results from layers 4, 5, 6, 8, 9, 10, timers. |
| Outputs | Workflow state transitions, proposed Actions, approval requests, escalations. |
| Must not | Call channel APIs. Authorise an Action itself; authorisation comes from the policy gate or a founder Approval. Skip a state. |

All transitions go through one transition function that checks the table in the [workflow document](workflows/investor-document-request.md#2-transitions) and writes the event in the same transaction.

MVP builds: the investor document request workflow, timers for reminders and expiry, the founder inbox read model.
Deferred: other workflow types, multi-step plans, delegation to people other than the founder.

### Layer 8: Execution Layer

| | |
| --- | --- |
| Responsibility | Perform authorised Actions through Integration. |
| Inputs | Action ID of an authorised Action. |
| Outputs | Execution result: succeeded with external reference, failed with error class, or outcome unknown. |
| Must not | Execute an Action whose authorisation is missing, stale or revoked. Alter recipient, body or attachment. Retry after an unknown outcome without reconciling first. |

Execution re-checks authorisation immediately before the side effect. It does not trust that Orchestration did.

MVP builds: send a reply with one attachment in an existing Gmail thread, with retry, backoff and reconciliation.
Deferred: scheduling meetings, follow-ups, other channels.

### Layer 9: Outcome Tracking Layer

| | |
| --- | --- |
| Responsibility | Verify what happened after execution and keep Conversation status current. |
| Inputs | Executed Action and its external reference; later channel observations. |
| Outputs | Outcome check results; Conversation status (awaiting reply, replied, bounced). |
| Must not | Infer success from the absence of an error. Mark a Workflow complete; it reports to Orchestration. |

MVP builds: confirm the sent message exists in the thread with the expected recipient and attachment; detect bounces within a fixed window; record when the Contact replies.
Deferred: follow-up proposals when no reply arrives, read receipts, meeting confirmation.

### Layer 10: Founder Intelligence Layer

| | |
| --- | --- |
| Responsibility | Hold what Axon knows about how one founder communicates and decides. |
| Inputs | Explicit preference settings; feedback signals (approvals, edits, rejections) from Orchestration. |
| Outputs | Preference profile for Intelligence (tone, length, sign-off). Suggested Permission changes for the founder to confirm. |
| Must not | Change a Permission. Loosen an autonomy tier. Apply an inferred preference to authorisation. |

Founder Intelligence is not a pipeline stage. Orchestration consults it when preparing inputs for Intelligence and writes feedback to it after founder decisions. Approval boundaries are stored as Permissions and enforced by the policy gate; Founder Intelligence may suggest them and the founder confirms them.

MVP builds: explicit preferences; storage of feedback signals.
Deferred: learning from feedback, style modelling from sent mail, suggested Permissions.

## 5. Cross-cutting capabilities

### A. Security & Governance

| Mechanism | Description |
| --- | --- |
| Authentication | Founder signs in to the web app; sessions are bound to one User and one Founder. Channel access uses OAuth with the narrowest scopes that support the workflow (see [`feasibility.md`](feasibility.md)). |
| Tenant isolation | Every table carries `founder_id`. PostgreSQL row-level security restricts each connection to the founder set for that request or job. |
| Policy gate | A deterministic function `evaluate(proposed_action, facts) → decision`. Inputs: action type, Contact, relationship category, verified flag, Document, interpretation confidence, the founder's Permissions. Output: tier, reason codes, matched Permission IDs, policy version. The most restrictive matching rule wins. With no matching Permission the tier is approval required. |
| Approval workflow | An Approval is a founder decision bound to a hash of the Action's recipient, body and attachment. A change to any of them voids it. |
| Authorisation check at execution | Execution refuses any Action that lacks a current policy decision of auto-execute or a valid Approval. |
| Encryption | TLS in transit. OAuth tokens encrypted at field level. Database and object store on encrypted volumes. |
| Audit log | The event log records every observation, proposal, decision, command and outcome with actor and provenance. It is append-only. |
| Retention and deletion | Raw items and message bodies expire on a schedule. A founder can delete all their data; events remain as content-free tombstones or are removed with the founder, per the data strategy in [`prd.md`](prd.md). |

### B. Reliability & Evaluation

| Mechanism | Description |
| --- | --- |
| Fake connector | Implements the connector contract in memory. Scenarios load a mailbox into it and assert on what was sent. |
| Fixture LLM provider | Returns recorded or scripted outputs so workflow and policy tests are deterministic. |
| Scenario runner | Feeds a scenario's messages through the real layers and compares resulting states, events, policy decisions and commands with expected values. Format and cases: [`evaluation.md`](evaluation.md). |
| Model evaluations | Run the real model over interpretation and drafting cases and score against expected outputs. Reported separately from deterministic tests. |
| Invariant tests | One or more automated tests per invariant in [`invariants.md`](invariants.md). |
| Regression rule | A production or pilot failure is added as a scenario before the fix is merged. |
| CI | Runs unit tests, invariant tests, the import-boundary check and the deterministic scenario suite on every change. |

### C. Observability & Monitoring

| Mechanism | Description |
| --- | --- |
| Trace ID | Assigned at ingestion and carried on every event, job, log line and LLM call for that Workflow. |
| Decision record | For each proposal: source message IDs, context item IDs, model ID, prompt version, interpretation, policy version, matched Permissions, reason codes. Answers why a decision was made. |
| Structured logs | JSON logs with trace ID, founder ID, layer and error class. Message content and credentials are excluded. |
| Workflow timeline | Web view that renders a Workflow's events in order. |
| Metrics | Per layer: latency, error count by class. Per connector: API errors and rate-limit responses. Per model call: latency, tokens, schema-validation failures. Queue depth and age of oldest job. |
| Alerts | Job stuck beyond its deadline, repeated connector authentication failures, a Workflow in a non-terminal state past its timeout. |

MVP builds: trace IDs, decision records, structured logs, the workflow timeline. Metrics dashboards and alerts are deferred until a pilot runs.

## 6. Stack

| Concern | Choice |
| --- | --- |
| Backend | Python, FastAPI |
| Database | PostgreSQL |
| Job queue and timers | PostgreSQL table polled by the worker |
| Frontend | Next.js, TypeScript |
| Local environment | Docker Compose: `api`, `worker`, `web`, `postgres` |
| LLM | Provider behind an `LLMProvider` interface |
| Blob storage | Object-store interface; local disk in the MVP |

Rationale and alternatives: [ADR 0001](adr/0001-stack.md), [ADR 0004](adr/0004-event-log-and-postgres-worker.md).

## 7. Contracts and versioning

- Events have a type, a schema version and a fixed envelope (see the [workflow LLD](workflows/investor-document-request.md#7-events)). A breaking change to an event creates a new version; consumers state which versions they accept.
- Layer interfaces are Python protocols with typed request and result objects. A layer's internals are private to its package.
- The connector contract is versioned as a whole. A connector declares the contract version it implements.
- The HTTP API is versioned under `/v1`.

## 8. Assumptions and open questions

Assumptions made to complete this document:

1. One backend deployable with separate `api` and `worker` processes, rather than one service per layer.
2. The policy gate lives in a governance module that Orchestration and Execution both call. It is not an eleventh layer.
3. With no matching Permission, the tier is approval required.
4. Founder Intelligence in the MVP stores explicit preferences and feedback only. It does not learn.
5. Context retrieval in the MVP uses structured queries. No embeddings.
6. Gmail ingestion polls. Push notifications are deferred.
7. A `Document` registry exists so the founder can state which files may be shared. The blueprint implies it but does not name it.

Open questions:

1. Which LLM provider and model to start with, and whether message content may be sent to a third-party model API under the pilot's privacy terms.
2. Hosting target for the pilot (affects key management and the object store).
3. Whether message bodies need field-level encryption in addition to volume encryption.
4. Retention periods for raw items and message bodies.
5. How the founder is notified of pending approvals outside the web app.
