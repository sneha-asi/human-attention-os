# Axon — Product Requirements Document

This document specifies what the first release of Axon does. Background is in the [one-pager](one-pager.md). Business objectives and metrics are in the [BRD](brd.md). How the system is built is in the [architecture](architecture.md). Terms such as Contact, Identity, Action, and Permission are defined in the [domain model](domain-model.md).

## 1. Users

The only user in this release is the Founder. Contacts receive email from Axon on the Founder's behalf and do not use the product.

## 2. MVP scope

One Founder, one Channel (Gmail), one Workflow: an investor emails the Founder asking for a financial document, and Axon handles the request through to a verified outcome.

| In scope | Out of scope |
| --- | --- |
| Connecting one Gmail account | Any other Channel |
| Recognising known Contacts by email address | Matching one Contact across Channels |
| Classifying a message as a financial-document request | Acting on other kinds of message |
| Retrieving prior Conversations with the Contact | Retrieving context from outside Gmail |
| A document library the Founder fills | Finding documents in Drive or the inbox |
| Permissions with three tiers | Permissions learned from behaviour |
| Drafting, approval, sending, and outcome verification | Sending follow-ups without the Founder's approval |
| A web app for setup, approvals, and history | Mobile app, voice |

Messages that are not financial-document requests from a Contact are left untouched in Gmail. Axon records that it saw them and takes no Action.

## 3. User workflows

### 3.1 Setup

1. The Founder signs in and connects Gmail.
2. The Founder adds Contacts, each with a name, an email address, and a relationship category. At least one has the category Investor.
3. The Founder adds documents to the library, each with a label (for example "Financial projections FY26") and a sensitivity mark.
4. The Founder sets Permissions. Each Permission names a relationship category or a single Contact, a document or document type, and a tier.
5. The Founder optionally writes style notes (for example "short, no greetings beyond first name").

### 3.2 Request that needs approval

1. An investor emails asking for financial projections.
2. Axon shows the Founder an approval item: the incoming message, the relevant history, what Axon understood, the draft reply, the attachment, and why approval is needed.
3. The Founder approves, edits and approves, or rejects.
4. On approval Axon sends the reply and confirms it was sent.
5. The item moves to history, and the Conversation stays open until the investor responds.

### 3.3 Pre-authorised request

Same as 3.2, without steps 2 and 3. The Founder sees the sent reply in history and is not interrupted.

### 3.4 Request that cannot proceed

Axon tells the Founder what stopped it: the sender is unknown, the document is missing, the request is unclear, the Permission is blocked, or the send failed. Nothing is sent to the Contact unless the Founder acts.

### 3.5 Review

The Founder can open any Action and see the source message, the classification, the Permission that applied, the approval, what was sent, and the outcome.

The valid states and transitions for these workflows are defined in the [workflow document](workflows/investor-document-request.md).

## 4. Functional requirements

### Connection and ingestion

| ID | Requirement |
| --- | --- |
| FR-1 | The Founder can connect one Gmail account through Google OAuth and disconnect it at any time. |
| FR-2 | Axon ingests each new inbound message on the connected account within 2 minutes of arrival. |
| FR-3 | A message delivered to Axon more than once produces one Message record and at most one Action. |
| FR-4 | After disconnection Axon ingests nothing and sends nothing on that account. |

### Contacts and identity

| ID | Requirement |
| --- | --- |
| FR-5 | The Founder can create, edit, and delete Contacts, each with one or more email Identities and one relationship category. |
| FR-6 | Axon matches an inbound message to a Contact by exact match of the sender address against stored Identities. |
| FR-7 | A message whose sender matches no Identity is marked as from an unknown sender. No reply is drafted or sent for it. |
| FR-8 | From an unknown-sender item, the Founder can add the sender as a new Contact or attach the address to an existing Contact. The message is then processed as from that Contact. |

### Understanding

| ID | Requirement |
| --- | --- |
| FR-9 | For each message from a known Contact, Axon produces a structured result: whether it is a financial-document request, which document is requested, a confidence value, and the text span it relied on. |
| FR-10 | If the confidence is below a configured threshold, or the request matches more than one library document, Axon asks the Founder to confirm the request and the document before drafting. |
| FR-11 | Axon retrieves prior Conversations with the Contact and identifies any open Commitment related to the request and any previous send of the same document. |
| FR-12 | The retrieved history shown to the Founder links each item to its source Message. |

### Documents and permissions

| ID | Requirement |
| --- | --- |
| FR-13 | The Founder can add, relabel, replace, and remove documents in the library and mark each as shareable or never-share. |
| FR-14 | The Founder can set a Permission for a relationship category or a single Contact, for a document or document type, with tier auto-execute, approval required, or blocked. |
| FR-15 | When more than one Permission applies, a Contact-level Permission overrides a category-level one. Among Permissions at the same level, the most restrictive applies. |
| FR-16 | When no Permission applies, the tier is approval required. |
| FR-17 | A document marked never-share is blocked for every Contact regardless of other Permissions. |
| FR-18 | The tier is computed by application code from stored Permissions. Model output is never an input to whether an Action is permitted. |

### Drafting and approval

| ID | Requirement |
| --- | --- |
| FR-19 | Axon drafts a reply in the same thread that addresses the request, attaches the matched document, and follows the Founder's style notes. |
| FR-20 | An approval item shows the incoming message, the history from FR-11, the classification, the draft, the attachment, the applicable Permission, and the reason approval is needed. |
| FR-21 | The Founder can approve, edit then approve, or reject an approval item. A rejection can carry a reason. |
| FR-22 | The message sent is exactly the content the Founder approved, including edits. |
| FR-23 | An approval applies to one Action. It does not change Permissions. |
| FR-24 | The Founder is notified by email when an item needs them. The notification contains no content from the incoming message or the draft. |
| FR-25 | If a new message arrives in the thread before a pending draft is approved, the pending item is withdrawn and the thread is processed again. |

### Execution

| ID | Requirement |
| --- | --- |
| FR-26 | Axon sends the reply through the Founder's Gmail account, in the original thread, only after the tier check passes and any required approval is recorded. |
| FR-27 | An Action results in at most one sent message, including after retries or restarts. |
| FR-28 | A blocked Action is never sent. The Founder is told which Permission blocked it. Approving a blocked Action is not possible. The Founder must change the Permission first. |
| FR-29 | If the requested document is not in the library, Axon sends nothing and asks the Founder to add it. Once it is added, the request continues from the tier check. |

### Outcome tracking

| ID | Requirement |
| --- | --- |
| FR-30 | After sending, Axon confirms that the message exists in the Gmail thread with the attachment, and records the Gmail message ID. Only then is the Action marked completed. |
| FR-31 | A transient send failure is retried up to a configured limit. After the limit, the Action is marked failed and the Founder is told. A failed Action is never shown as completed. |
| FR-32 | Axon records when the Contact replies in the thread and marks the Conversation as responded. |
| FR-33 | If the Contact has not replied after a configured number of days, Axon proposes a follow-up to the Founder. A follow-up always requires approval in this release. |

### Founder preferences

| ID | Requirement |
| --- | --- |
| FR-34 | The Founder can write and edit style notes that are applied to every draft. |
| FR-35 | Axon records every edit and rejection the Founder makes. In this release these records do not change drafting or Permissions automatically. |

### Audit and data control

| ID | Requirement |
| --- | --- |
| FR-36 | Every Action has a record containing the source Message, the classification result, the model and prompt version, the Permission evaluated, the approval if any, the content sent, and the outcome. Records cannot be edited. |
| FR-37 | The Founder can view the history of all Actions and filter by Contact and by status. |
| FR-38 | The Founder can delete their account. Stored messages, documents, Contacts, and Permissions are deleted within a configured period. |

## 5. Non-functional requirements

| ID | Requirement |
| --- | --- |
| NFR-1 | Every data access is scoped to one Founder. An automated test attempts cross-Founder reads and writes on every data-access path and must fail to obtain data. |
| NFR-2 | Data is encrypted in transit with TLS and at rest. OAuth tokens are stored encrypted and never written to logs. |
| NFR-3 | Axon stores message content only for threads involving a known Contact. For other messages it stores the Gmail message ID, the sender address, and the time. |
| NFR-4 | No code path sends a message without passing the tier check. An automated architectural test fails the build if one is added. |
| NFR-5 | The time from a message arriving in Gmail to an approval item or an auto-sent reply is under 5 minutes at the 95th percentile, excluding time waiting for the Founder. |
| NFR-6 | Gmail and model-provider outages delay Actions and do not lose them. Work in progress resumes after a restart without duplicating a send. |
| NFR-7 | Every external call has a timeout and a bounded retry policy. |
| NFR-8 | Each Action can be traced through every processing step by a single trace ID. Latency, failures, and model calls are recorded for each step. |
| NFR-9 | The model provider is called through one interface. Changing provider requires no change outside that module. |
| NFR-10 | The evaluation set runs on every change to prompts, models, or decision logic. A release does not ship if a previously passing case fails. See [evaluation](evaluation.md). |
| NFR-11 | The web app works in current versions of Chrome and Safari on desktop and on a phone-width screen. |
| NFR-12 | Founder data is sent to a model provider only under terms that prohibit training on it. |

The complete list of rules that must always hold is in [invariants](invariants.md).

## 6. Acceptance criteria

The cases below use one setup. The Founder is Asha. Her Gmail is connected. Ravi (ravi@northfund.example) is a Contact with category Investor. The library holds "Financial projections FY26", marked shareable. Unless stated, there is no Permission for it, so the default tier applies.

### AC-1 Approval required

- **Given** the setup above,
- **when** Ravi emails "Could you send over the updated projections before Friday?",
- **then** within 5 minutes Asha has an approval item showing Ravi's message, the prior thread history, the classification "financial-document request: Financial projections FY26", a draft reply with the document attached, and the reason "no permission set; approval required by default";
- **and** nothing has been sent to Ravi.
- **When** Asha approves,
- **then** one reply is sent in the same thread with the attachment, the Action is completed with a Gmail message ID, and the audit record shows Asha's approval.

### AC-2 Edit before approval

- **Given** the approval item from AC-1,
- **when** Asha changes the draft text and approves,
- **then** the sent email matches her edited text exactly, and the edit is recorded.

### AC-3 Rejection

- **Given** the approval item from AC-1,
- **when** Asha rejects it with the reason "will send after board meeting",
- **then** nothing is sent, the Action is closed as rejected with the reason, and no further prompt is raised for that message.

### AC-4 Pre-authorised auto-send

- **Given** a Permission "Investor, Financial projections FY26, auto-execute",
- **when** Ravi sends the same email,
- **then** the reply is sent with the attachment without any approval item or notification;
- **and** the Action appears in history as completed, with the Permission named as the authorisation.

### AC-5 Unknown sender

- **Given** no Contact has the address maya@otherfund.example,
- **when** that address sends "Please share your financial projections",
- **then** no draft is created and nothing is sent;
- **and** Asha sees an unknown-sender item offering to add the sender as a Contact.
- **When** Asha adds Maya as an Investor,
- **then** the message is processed as in AC-1.

### AC-6 Document not available

- **Given** the library has no document matching the request,
- **when** Ravi asks for "the Q3 cap table",
- **then** nothing is sent, and Asha sees an item stating that the requested document is not in the library.
- **When** Asha adds the document,
- **then** the request continues from the tier check and behaves as in AC-1.

### AC-7 Blocked by permission

- **Given** a Permission "Ravi, Financial projections FY26, blocked" and a Permission "Investor, Financial projections FY26, auto-execute",
- **when** Ravi requests the projections,
- **then** nothing is sent, Asha is told the request was blocked and by which Permission, and the item has no approve control.

### AC-8 Blocked by never-share

- **Given** "Financial projections FY26" is marked never-share and a Permission "Investor, Financial projections FY26, auto-execute" exists,
- **when** Ravi requests the projections,
- **then** nothing is sent and the Action is recorded as blocked.

### AC-9 Send failure

- **Given** AC-1 up to Asha's approval, and the Gmail send call fails on every attempt,
- **when** the retry limit is reached,
- **then** the Action is marked failed, Asha is told the reply was not sent, and the Action is not shown as completed anywhere.
- **When** Asha chooses to retry and Gmail is available,
- **then** exactly one reply is sent.

### AC-10 No duplicate send

- **Given** AC-4, and the process is restarted after the Gmail send call succeeds but before the outcome is recorded,
- **when** processing resumes,
- **then** Ravi's thread contains exactly one reply from Axon.

### AC-11 Duplicate delivery

- **Given** Gmail delivers the notification for Ravi's email twice,
- **then** one Message record, one Action, and at most one approval item exist.

### AC-12 Unclear request

- **Given** Ravi writes "Can you send me the numbers we discussed?" and the library holds two financial documents,
- **then** Asha is asked to confirm which document is meant before any draft is shown, and nothing is sent.

### AC-13 Message out of scope

- **Given** Ravi writes "Great meeting you yesterday, let's catch up next month",
- **then** no Action is created and Asha receives no notification.

### AC-14 Thread changes while approval is pending

- **Given** the approval item from AC-1 is pending,
- **when** Ravi sends a second message in the thread saying "Actually, hold off, send it next week",
- **then** the pending item is withdrawn and cannot be approved, and the thread is processed again with both messages.

### AC-15 Follow-up proposal

- **Given** AC-1 completed and Ravi has not replied after the configured number of days,
- **then** Asha receives a follow-up proposal that requires her approval, and no follow-up is sent without it.

### AC-16 Audit

- **Given** any completed, rejected, blocked, or failed Action from the cases above,
- **when** Asha opens it,
- **then** she sees the source message, the classification, the Permission evaluated, the approval or its absence, the content sent if any, and the outcome.

### AC-17 Disconnection

- **Given** Asha disconnects Gmail while an approval item is pending,
- **then** the item is withdrawn, nothing is sent, and no new messages are ingested.

## 7. Assumptions and open questions

### Assumptions made in this document

- **Founder surface.** Approvals happen in a web app, with an email notification. The blueprint does not say where the Founder interacts with Axon.
- **Contacts are entered by the Founder.** Axon does not infer who is an investor from the inbox in this release.
- **Document library.** The Founder uploads the documents Axon may send. Axon does not search Drive or past attachments.
- **Permission shape.** A Permission is (category or Contact) × (document or document type) × tier. Contact-level overrides category-level, and the most restrictive wins at the same level. The default is approval required.
- **Blocked means not approvable.** The Founder must change the Permission to proceed. This keeps the blocked tier distinct from approval required.
- **Founder Intelligence in the MVP** is explicit style notes and Permissions. Edits and rejections are recorded for later use and change nothing automatically.
- **Follow-ups always need approval** in this release.
- **Out-of-scope messages are ignored silently.** Axon does not triage the rest of the inbox.
- **Numeric values** in FR-2 (2 minutes) and NFR-5 (5 minutes) are proposals. Retry limits, the confidence threshold, the follow-up interval, and the deletion period are left as configuration.

### Open questions

1. Should approvals also be possible by replying to the notification email, or from a phone app?
2. Is the 5-minute latency in NFR-5 acceptable for an auto-send, or would founders prefer a deliberate delay so replies do not look automated?
3. Should auto-sent replies be disclosed to the recipient as sent by an assistant?
4. How many days without a reply should trigger a follow-up proposal?
5. Should Axon propose Contacts and categories from the inbox during setup to shorten onboarding?
6. Should a pending approval expire, and what happens to the request if it does?
7. Which document types beyond financial projections belong in the first release, if any?
8. What retention period applies to stored message content while the account is active?
