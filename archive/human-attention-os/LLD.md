# Human Attention OS — Low-Level Design

## 1. Engineering thesis

The product is a **permissioned, event-driven execution system** that converts multilingual human communication into durable organizational state and moves routine work forward within policy.

The hard engineering problems are:

- understanding intent across voice, text, languages, and code-mixing;
- resolving people, projects, commitments, and deadlines against company context;
- deciding whether human attention is required;
- executing actions reliably across imperfect external systems;
- learning preferences without silently changing safety boundaries;
- proving why every action was taken.

## 2. System shape

```text
                           ┌──────────────────────┐
Voice / text / integrations│   Ingestion Gateway   │
──────────────────────────▶│ normalize + authorize │
                           └──────────┬───────────┘
                                      │ events
                           ┌──────────▼───────────┐
                           │  Conversation Ledger  │
                           │ immutable provenance  │
                           └──────────┬───────────┘
                                      │
                 ┌────────────────────▼────────────────────┐
                 │         Context & Semantic Runtime        │
                 │ identity · entities · memory · policies   │
                 └───────────────┬───────────────┬───────────┘
                                 │               │
                 ┌───────────────▼──────┐ ┌────▼────────────────┐
                 │ Intent / Commitment  │ │ Attention Firewall   │
                 │ extraction + linking │ │ risk + human need   │
                 └───────────────┬──────┘ └────┬────────────────┘
                                 │             │
                           ┌─────▼─────────────▼─────┐
                           │   Action & Policy Graph  │
                           │ state, dependencies, SLA│
                           └──────────────┬──────────┘
                                          │ commands
                           ┌──────────────▼──────────┐
                           │ Durable Workflow Runtime │
                           │ retries, timers, idempot. │
                           └──────────────┬──────────┘
                                          │
                           ┌──────────────▼──────────┐
                           │ Connector / Tool Runtime │
                           │ Slack · email · tasks    │
                           └──────────────┬──────────┘
                                          │ events
                           ┌──────────────▼──────────┐
                           │ Exception Inbox + Voice  │
                           │ human review + feedback  │
                           └─────────────────────────┘
```

## 3. Core design principles

### 3.1 Event-sourced operational memory

Store immutable events and derive current state from them. Nothing that happened is overwritten.

Examples:

- `message.received`
- `audio.transcribed`
- `language.detected`
- `commitment.proposed`
- `commitment.confirmed`
- `action.assigned`
- `action.reminded`
- `action.completed`
- `action.escalated`
- `human.override_recorded`

Every derived action must retain its source event IDs, model version, prompt/policy version, confidence, and actor.

### 3.2 Commands are separate from observations

The system must distinguish:

- **Observations:** what was said, detected, or found in an external system.
- **Proposals:** what the AI recommends.
- **Commands:** what the system is authorized to do.
- **Events:** what actually happened.

No model output directly performs an external side effect. All side effects pass through authorization and command execution.

### 3.3 Policy before intelligence

The model may suggest an action. The policy engine decides whether that action is permitted.

```text
Model suggestion → policy evaluation → risk classification → approval gate → command
```

### 3.4 Uncertainty is stored, not resolved by guessing

When the input is ambiguous, the system records the ambiguity and its candidates instead of picking one.

Examples:

- multiple possible owners;
- relative date without a reference timezone;
- unclear whether a statement is a commitment;
- conflicting company context;
- low-confidence language or transcription.

## 4. Service boundaries

### 4.1 Ingestion Gateway

Responsibilities:

- receive text, audio, transcripts, webhooks, and connector events;
- authenticate source and actor;
- enforce source permissions;
- assign tenant, workspace, and correlation IDs;
- normalize inputs to a canonical envelope;
- publish immutable ingestion events.

Canonical envelope:

```json
{
  "event_id": "evt_01",
  "event_type": "message.received",
  "workspace_id": "sarvam",
  "actor_id": "person_123",
  "source": "slack",
  "source_object_id": "slack_msg_456",
  "occurred_at": "2026-09-16T10:00:00Z",
  "payload_ref": "blob://secure/input/evt_01",
  "language_hint": null,
  "trace_id": "trace_01"
}
```

### 4.2 Sarvam Language Runtime

An abstraction over Sarvam capabilities.

Responsibilities:

- speech-to-text and audio normalization;
- language identification;
- code-mix and script handling;
- translation and transliteration when needed;
- text-to-speech response generation;
- structured intent extraction;
- model selection by language, task, latency, and cost;
- model fallback and degraded-mode behavior.

The runtime must record model, version, language, latency, token/audio cost, and quality metadata for every call.

### 4.3 Context Runtime

Maintains the organization’s operational context.

Core entities:

- `Person`
- `Role`
- `Team`
- `Project`
- `Priority`
- `Policy`
- `Workflow`
- `Relationship`
- `Preference`
- `SourcePermission`

Context retrieval must be permission-aware and source-grounded. Every retrieved fact includes provenance, freshness, visibility, and confidence.

### 4.4 Semantic Runtime

Converts communication into structured meaning.

Pipeline:

```text
raw input
→ transcription / normalization
→ language and speaker metadata
→ entity resolution
→ intent classification
→ commitment extraction
→ deadline normalization
→ owner resolution
→ dependency detection
→ uncertainty calculation
→ proposal generation
```

The semantic runtime must support partial results and human correction. A correction should update the specific semantic field, not rewrite the entire record.

### 4.5 Attention Firewall

Determines whether an item should be automated, delegated, batched, deferred, declined, or surfaced to a human.

Inputs:

- action type;
- value and urgency;
- risk and reversibility;
- user role and responsibility;
- personal preferences;
- organizational policies;
- confidence and ambiguity;
- current workload and deadlines;
- relationship and communication sensitivity.

Output:

```json
{
  "route": "auto_execute",
  "risk_tier": "low",
  "human_required": false,
  "reason_codes": ["internal", "reversible", "high_confidence"],
  "policy_version": "policy_07",
  "expires_at": "2026-09-16T18:00:00Z"
}
```

The output is a routing decision, not an external action.

### 4.6 Action Graph

Represents work as a graph rather than a flat task list.

Nodes:

- commitments;
- actions;
- decisions;
- people;
- projects;
- messages;
- approvals;
- evidence;
- escalations.

Edges:

- `owned_by`
- `depends_on`
- `derived_from`
- `blocks`
- `requires_approval`
- `evidenced_by`
- `escalates_to`
- `related_to`

This enables the system to answer: “What is blocked, why, by whom, and what is the smallest action that resolves it?”

### 4.7 Durable Workflow Runtime

Executes approved action plans with:

- idempotency keys;
- retries with backoff;
- timeout handling;
- durable timers;
- dead-letter queues;
- compensating actions;
- connector rate-limit handling;
- human approval pauses;
- full command and result history.

### 4.8 Connector Runtime

All external systems use a common connector contract:

```text
authorize()
capabilities()
validate(command)
execute(command, idempotency_key)
observe(external_object)
reconcile()
```

Initial connector types:

- messaging;
- email;
- task/project system;
- calendar;
- document/evidence store.

Connectors must support reconciliation because external systems can change independently of our system.

### 4.9 Exception Inbox

The inbox is a prioritized queue of decisions that need a human.

Each item contains:

- the decision required;
- why it reached the human;
- relevant source context;
- proposed next action;
- confidence and risk;
- deadline or SLA;
- affected people and projects;
- available approve/edit/reject/delegate actions.

## 5. Data model

### Commitment

```json
{
  "id": "commitment_01",
  "workspace_id": "sarvam",
  "statement": "Send the enterprise demo proposal",
  "normalized_action": "send_proposal",
  "owner_id": "person_123",
  "requester_id": "person_456",
  "project_id": "project_789",
  "due_at": "2026-10-02T12:00:00+05:30",
  "status": "confirmed",
  "priority": "high",
  "confidence": 0.91,
  "source_event_ids": ["evt_01"],
  "evidence_ids": [],
  "policy_version": "policy_07",
  "created_at": "2026-09-16T10:00:00Z",
  "updated_at": "2026-09-16T10:01:00Z"
}
```

### Action state machine

```text
PROPOSED
  ↓
NEEDS_CLARIFICATION ──→ PROPOSED
  ↓
NEEDS_CONFIRMATION ────→ APPROVED
  ↓                         ↓
REJECTED                 EXECUTING
                            ↓
                    WAITING_FOR_EVIDENCE
                       ↓           ↓
                   COMPLETED    BLOCKED
                                    ↓
                                ESCALATED
```

Every transition is an event and must be authorized.

### Learning record

```json
{
  "id": "learning_01",
  "scope": "person",
  "subject_id": "person_123",
  "preference": "routine_internal_followups",
  "value": "auto_execute",
  "basis": "explicit_instruction",
  "confidence": 1.0,
  "status": "active",
  "created_from_event": "evt_99"
}
```

Learned preferences cannot override organization policy or safety rules.

## 6. Decision engine

The decision engine combines deterministic rules with model proposals.

```text
1. Is the source and actor authorized?
2. What is the normalized intent?
3. What entities and commitments are involved?
4. Who is responsible according to context?
5. What is the risk and reversibility?
6. Does policy permit this action?
7. Is confidence above the action threshold?
8. Does the user preference permit autonomy?
9. Does the action require human approval?
10. Route, explain, and log the decision.
```

Decision result:

```json
{
  "decision": "needs_confirmation",
  "action_type": "send_internal_followup",
  "risk_tier": "medium",
  "confidence": 0.86,
  "required_approver": "person_456",
  "reason_codes": ["deadline_inferred", "shared_project"],
  "expires_at": "2026-09-17T10:00:00Z"
}
```

## 7. API surfaces

### Ingest an input

```http
POST /v1/inputs
```

Creates an authorized input event. The API returns an ingestion ID and does not synchronously execute actions.

### List attention items

```http
GET /v1/attention?status=open&assignee=me
```

Returns only items routed to the caller by the Attention Firewall.

### Approve or edit a proposal

```http
POST /v1/proposals/{proposal_id}/decision
```

Body:

```json
{
  "decision": "approve",
  "edits": {},
  "reason": null
}
```

### Voice command

```http
POST /v1/voice/commands
```

Returns a structured proposal, not an immediate side effect.

### Action status

```http
GET /v1/actions/{action_id}
```

Returns current state, provenance, decisions, external references, evidence, and audit history.

## 8. Reliability requirements

- No duplicate external messages for one logical command.
- All external commands use idempotency keys.
- External state is periodically reconciled.
- Failed actions enter a visible retry or human-review state.
- A model outage must not lose committed work.
- A connector outage must pause safely and resume later.
- Every automated action is explainable after the fact.
- Sensitive payloads are minimized and encrypted.
- Audio and transcripts have explicit retention policies.

## 9. Evaluation strategy

Build a labeled internal evaluation set covering:

- clear commitments;
- implied commitments;
- non-commitments;
- multiple owners;
- ambiguous deadlines;
- code-mixed speech;
- different Indian languages;
- sarcasm and conversational noise;
- conflicting context;
- sensitive actions;
- cases where no action is required.

Measure separately:

- transcription quality;
- language identification;
- entity resolution;
- commitment precision and recall;
- owner accuracy;
- deadline accuracy;
- risk classification;
- attention-routing accuracy;
- action execution reliability;
- user correction rate.

The primary metric is the percentage of routine work completed correctly without human intervention, with zero unauthorized actions.

## 10. Build sequence

### Slice 1 — Read-only semantic pipeline

Input → Sarvam transcription/language processing → commitment proposals → provenance and evaluation UI.

### Slice 2 — Context and attention routing

Add people, teams, permissions, policy rules, risk tiers, and exception inbox.

### Slice 3 — Confirmed execution

Add one messaging connector and one task connector with approvals, idempotency, and audit logs.

### Slice 4 — Durable follow-through

Add reminders, timers, evidence checks, reconciliation, and escalation.

### Slice 5 — Bounded autonomy and learning

Add explicit preferences, behavior-based suggestions, confidence thresholds, and safe auto-execution.

The first slice to build end to end is specified in [`BUILD_SPEC.md`](BUILD_SPEC.md). It succeeds when a real cross-functional commitment moves from conversation to closure with less human coordination and a complete record of why the system acted.
