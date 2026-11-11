# Axon — Data Strategy

What Axon stores, what it retrieves, what leaves the system, and when data is removed. This document is the index for those rules. The binding statements are the requirements and invariants it cites: [PRD](prd.md), [invariants](invariants.md), [architecture](architecture.md) Layer 3 and section 5A, and the [domain model](domain-model.md).

## 1. What is stored

| Data | What is kept | Kept until | Rule |
| --- | --- | --- | --- |
| Gmail credentials | OAuth tokens, encrypted at field level | Founder deletion | NFR-2, INV-3 |
| Messages from a known Contact | Metadata row, plus the body and raw item as blobs | Body and raw item: retention expiry. Metadata: deletion of the Conversation or the Founder | NFR-3, domain model (Message) |
| Messages from anyone else | Gmail message ID, sender address and time. No content | Founder deletion | NFR-3, FR-7 |
| Attachments on stored Messages | Blob, filename, type, size, hash | With the Message body | Domain model (Attachment) |
| Documents in the Founder's library | Blob, title, kind, version, hash, sharing mark | The Founder removes them, or Founder deletion | FR-13 |
| Contacts, Identities, Permissions, style notes | As entered by the Founder | The Founder deletes them, or Founder deletion | FR-5, FR-14, FR-34 |
| Decision record for each Action | Interpretation, Policy Decision, Approval, content sent, Outcome Checks, model and prompt version | Founder deletion. Not editable | FR-36 |
| Founder edits and rejections | The edit or the rejection reason | Founder deletion | FR-35 |
| Event log | Event type, IDs and references. No message content | Founder deletion. Append-only | INV-3, INV-17 |
| Application logs | IDs, timings, error classes. No bodies, attachments or credentials | Not yet specified | INV-3 |

Retention periods are not set ([open question 16](open-questions.md)). Because events and decision records hold references, expiring a message body does not require rewriting them.

## 2. What is retrieved

For each incoming Message, Context & Memory returns the current Conversation, recent Conversations with the same Contact, open Tasks and Commitments, earlier Actions toward the Contact, and candidate Documents. Retrieval is a structured query bounded by count and age, and every item carries the ID of its source (FR-11, FR-12). Semantic search and long-term summaries are deferred.

## 3. Isolation

Every table carries `founder_id`, and PostgreSQL row-level security limits each request or job to one Founder. Blob paths are prefixed by Founder ID. A test calls every data-access path as one Founder with another's IDs and must get nothing (INV-1, NFR-1).

## 4. What leaves the system

| Destination | What is sent | Constraint |
| --- | --- | --- |
| Gmail | The approved or pre-authorised reply and its attachment, in the original thread | FR-26; only through the Execution Layer |
| Model provider | Message content and retrieved history needed to interpret a request and draft a reply. Never credentials | NFR-12, INV-3. Provider and terms undecided ([open question 15](open-questions.md)) |
| Founder's notification email | A notice that an item is waiting. No content from the message or the draft | FR-24 |

Nothing else leaves. Logs and events contain no content, so they can be shipped to monitoring tools without carrying founder data.

## 5. Deletion

| Trigger | Effect | Rule |
| --- | --- | --- |
| Retention expiry | Message bodies, raw items and their attachment blobs are deleted. Metadata rows remain | INV-4, architecture Layer 3 |
| The Founder removes a Contact, Document or Permission | That record is deleted or archived | FR-5, FR-13, FR-14 |
| The Founder disconnects Gmail | Ingestion and sending stop. Stored data is unaffected | FR-4; see open question 31 |
| The Founder deletes their account | All rows and blobs for that Founder are removed. A tombstone with no content may remain | FR-38, INV-4 |

## 6. Assumptions

These are stated here because no requirement covers them yet. Each is in [open questions](open-questions.md).

- The Founder owns their data. Axon uses it only to serve that Founder and does not combine it with another Founder's for any purpose, including evaluation and preference learning (question 30).
- Disconnecting Gmail does not delete stored content (question 31).
- Evaluation fixtures are synthetic. Real founder data is used in the pilot only with consent and is not copied into test fixtures ([evaluation](evaluation.md), [`AGENTS.md`](../AGENTS.md)).
