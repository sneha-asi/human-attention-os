# Axon — System Invariants

Rules that hold in every state of the system. Each has an enforcement mechanism in the code or database and at least one automated test. A change that breaks an invariant test is not merged.

Terms are defined in [`domain-model.md`](domain-model.md). States and status values are defined in the [workflow document](workflows/investor-document-request.md).

## Isolation and data

### INV-1 Founder isolation

No query, job, prompt or response contains data belonging to a Founder other than the one it runs for.

- Enforcement: every table has `founder_id`. PostgreSQL row-level security filters on a per-connection setting that the request or job sets once. The application database role cannot bypass row-level security. Blob paths are prefixed by Founder ID and resolved through Storage.
- Test: seed two Founders. For every repository method and every HTTP endpoint, call it as Founder A with IDs belonging to Founder B and assert an empty result or `404`. A schema test asserts every table has `founder_id` and an enabled policy.

### INV-2 One Message per provider message

A provider message is stored once per Channel Account, however many times it is delivered.

- Enforcement: unique constraint on `(channel_account_id, provider_message_id)`. Ingestion uses insert-or-ignore and emits `message.ingested` only on insert.
- Test: feed the same raw item three times through the fake connector. Assert one Message, one `message.ingested` event and one Workflow.

### INV-3 Credentials and content stay out of logs, events and prompts

OAuth tokens never appear in logs, events or LLM prompts. Message bodies and attachment content never appear in logs or events.

- Enforcement: tokens are stored encrypted and exposed only to Integration through a type that does not serialise. Event payload schemas contain IDs and references only. The logger rejects fields outside an allow-list.
- Test: run the scenario suite with a marker string as the token and another in every message body. Scan captured logs and the `events` table for either marker; scan recorded prompts for the token marker.

### INV-4 Deletion is complete

After a Founder's data is deleted, no row or blob for that Founder remains, apart from a tombstone that contains no content.

- Enforcement: deletion runs as one job that removes rows by `founder_id` in dependency order and deletes the blob prefix. Retention expiry clears `body_ref` and `raw_ref` and deletes the blobs.
- Test: run a full scenario, delete the Founder, then assert zero rows per table for that `founder_id` and an empty blob prefix.

## Authorisation

### INV-5 No side effect without an authorised Action

A channel side effect happens only for an Action in status `AUTHORISED` whose authorisation is current.

- Enforcement: `Connector.execute` is callable only from the Execution package (import-boundary check). Execution loads the Action and calls the policy gate's `check_authorisation` in the same transaction that moves it to `EXECUTING`. The fake and real connectors reject a command without an authorisation reference.
- Test: attempt to execute Actions in every status other than `AUTHORISED`, and an `AUTHORISED` Action with a revoked Permission, a mismatched `content_hash` and a missing Approval. Assert no connector call in each case.

### INV-6 Authorisation comes from the policy gate or the founder

An Action becomes `AUTHORISED` only through a Policy Decision with tier `auto_execute` or an Approval with decision `approved` by an `owner` User. Model output cannot authorise. The two model-derived facts the gate reads, confidence and the content-risk flag, can only make the tier more restrictive.

- Enforcement: one function performs the `PROPOSED → AUTHORISED` change and requires one of the two records. The policy gate is pure code with no LLM call. The Interpretation schema has no tier or authorisation field.
- Test: property test over generated Interpretations, including ones with injected instructions and extra fields: the resulting tier depends only on the facts passed to the policy gate. A static check asserts the governance package does not import the LLM provider.

### INV-7 Tier resolution is fixed, and blocked cannot be approved

A Document marked `never_share` gives `blocked` whatever Permissions exist. Otherwise, Permissions that name the Contact take precedence over Permissions that name its relationship; among Permissions at the same level the most restrictive wins; no match gives `approval_required`. A blocked Action cannot be approved.

- Enforcement: tier resolution is one function that applies these steps in order and takes the maximum over an ordered enum within a level. The approve endpoint re-runs the policy gate and refuses when the tier is `blocked`.
- Test: table-driven tests over Permission combinations, including a Contact-level `auto_execute` with a relationship-level `blocked`, and a `never_share` Document with an `auto_execute` Permission. An approve request against a blocked Action returns an error and leaves the Workflow in `BLOCKED`.

### INV-8 Auto-execute requires a verified, known sender and a confident, clean interpretation

An Action is never auto-executed when the triggering Message's sender is unverified, the sender is not a Contact, the reply address differs from the stored Identity, interpretation confidence is below the threshold, or the Interpretation reports instructions addressed to Axon inside the Message. A Message from an unknown sender starts no Workflow at all; the Contact condition here is a second check.

- Enforcement: these facts are inputs to the policy gate, which downgrades `auto_execute` to `approval_required` and adds a reason code.
- Test: for each condition, start from a scenario that auto-executes, flip the one fact, and assert `DECISION_PENDING` with the expected reason code.

### INV-9 An Approval covers exactly what the founder saw

An Approval is valid only for the Action `content_hash` it was given for. Changing the recipient, body or attachment voids it.

- Enforcement: the Approval stores the `content_hash`. `check_authorisation` compares it to the Action's current hash. Execution recomputes the hash from the bytes it is about to send.
- Test: approve an Action, modify each of the three components in turn, and assert execution is refused and the Workflow returns to `UNDERSTOOD`.

### INV-10 Model output does not choose recipients or file bytes

The recipient of an outbound message comes from the stored Identity or the verified sender address of the triggering Message. The attachment is a registered Document selected by ID. The model supplies only the reply body and a Document ID from the list it was given.

- Enforcement: the Action builder ignores any address in model output and rejects a Document ID that was not in the candidate list.
- Test: fixture LLM returns a draft containing another address and a Document ID outside the candidate list. Assert the recipient is unchanged and the Workflow escalates with `invalid_document`.

### INV-11 Learned preferences do not change Permissions

Only an `owner` User can create, change or revoke a Permission. Founder Intelligence can suggest.

- Enforcement: Permission writes exist only in the governance module behind an endpoint that requires an `owner` session. The Founder Intelligence package has no write access to Permissions (import-boundary check).
- Test: record feedback signals that would justify a looser tier; assert the Permission set is unchanged. Static import check.

## Execution and outcomes

### INV-12 Layer boundaries hold

Only Integration calls channel APIs. Only Intelligence calls the LLM provider. Layers import only what the call rules in [`architecture.md`](architecture.md#3-call-rules) allow.

- Enforcement: import-boundary contracts checked in CI. Channel and LLM client libraries are dependencies of those two packages only.
- Test: the import check itself, plus a test that adds a forbidden import in a temporary module and asserts the check fails.

### INV-13 At most one send per Action

One Action results in at most one message on the channel, across retries, crashes and duplicate jobs.

- Enforcement: unique `idempotency_key` per Action. Execution takes a row lock, uses a two-step create-then-send with the intermediate reference stored before sending, and reconciles any `OUTCOME_UNKNOWN` before another attempt. See [the workflow LLD](workflows/investor-document-request.md#9-idempotency).
- Test: fake connector injects a timeout after a successful send, a crash between the two steps, and duplicate job delivery. Assert exactly one sent message in each case.

### INV-14 Success is recorded only with evidence

An Action is `SUCCEEDED` only when the connector returned an external reference or reconciliation found the sent message. A Workflow is `COMPLETED` only when an Outcome Check with result `pass` exists for its Action.

- Enforcement: the functions that set these values require the evidence record as an argument. A database check constraint requires `external_ref` when `status = 'SUCCEEDED'`.
- Test: drive a Workflow with the outcome check failing or inconclusive and assert it never reaches `COMPLETED`. Attempt to insert a `SUCCEEDED` Action without `external_ref` and assert the constraint fails.

### INV-15 Failures stay failures

An error, timeout or unknown outcome is recorded as such. Terminal Workflow states and terminal Action statuses do not change.

- Enforcement: the transition table has no transitions out of terminal states. `OUTCOME_UNKNOWN` can move only through reconciliation. Exception handlers in the worker record the error class and never set a success status.
- Test: for each terminal state, attempt every transition and assert rejection. Inject each connector error class and assert the recorded status and event.

### INV-16 Transitions follow the table and are recorded

A Workflow changes state only through a transition listed in its workflow document, and each change writes a `workflow.state_changed` event in the same transaction.

- Enforcement: one transition function is the only writer of `workflows.state`. It validates against the table and inserts the event before commit.
- Test: exhaustive test of state pairs against the table. After any scenario, replaying the Workflow's events reproduces its current state.

### INV-17 The event log is append-only

Events are never updated or deleted, except by Founder deletion (INV-4).

- Enforcement: the application database role has `INSERT` and `SELECT` on `events` only. Deletion runs under a separate role.
- Test: attempt `UPDATE` and `DELETE` on `events` with the application role and assert a permission error.

### INV-18 Every Action has provenance

Every Action can be traced to its triggering Message, Interpretation, Policy Decision and, where applicable, Approval.

- Enforcement: non-null foreign keys from Action to Workflow, and from Workflow to trigger Message. Authorisation requires a Policy Decision. The Interpretation records the context item IDs it was given.
- Test: after each scenario, walk from every Action to its sources and assert each link resolves.

### INV-19 One active Workflow per Conversation

A Conversation has at most one Workflow in a non-terminal state.

- Enforcement: partial unique index on `workflows(conversation_id)` where the state is non-terminal. A new inbound Message either supersedes the active Workflow or waits for it (see the workflow document).
- Test: deliver two Messages in one thread in quick succession and assert one active Workflow and no duplicate replies.

## Assumptions and open questions

Assumptions:

1. The four example invariants in the blueprint are covered by INV-1, INV-5, INV-14 and INV-15. The rest are derived from the blueprint's engineering principles and the first workflow.
2. The confidence threshold in INV-8 is a configuration value. Its starting value is set from evaluation results ([`evaluation.md`](evaluation.md)).

Open questions are collected in [`open-questions.md`](open-questions.md).
