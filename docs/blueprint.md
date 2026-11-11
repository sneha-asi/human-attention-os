# Axon — Blueprint

This is the top-level plan: what Axon is, how it is structured, and the order of work. Detail lives in the documents linked from each section. Section 10 lists what changed from the original blueprint and why.

## 1. What Axon is

Axon manages a founder's professional communication. It reads incoming messages, works out what each one asks for, and acts within permissions the founder has set. It asks the founder only when the decision is theirs.

The reasoning: model capability keeps getting cheaper, and a person's attention does not. Today a founder reads every message and then delegates some of the work to AI. Axon reverses the order: the system reads first and the founder sees what needs them.

Communication is the starting point. Automating execution beyond communication, and for people other than founders, comes later and is not designed here. See the [one-pager](one-pager.md).

## 2. Why founders first

- They receive a high volume of professional messages across investors, customers, staff and vendors.
- Many of those messages carry consequences, so a system that handles them has to get permissions right from the start.
- Much of the volume is repetitive: follow-ups, scheduling, document requests, status questions.
- One person decides what Axon may do, which keeps the permission model simple.

## 3. What Axon competes on

Axon uses existing models and APIs. Its advantage has to come from things that accumulate with use and do not depend on which model is underneath:

- context about each contact and conversation;
- each founder's stated permissions and preferences;
- a record of actions taken and their verified outcomes;
- measured reliability on a growing evaluation set.

## 4. Channels

The channel list is limited by what providers allow. The table summarises [feasibility](feasibility.md), which was researched from search excerpts of provider pages; confirm any item on the provider's page before committing to it.

| Order | Channel | Position |
| --- | --- | --- |
| 1 | Gmail | First channel. Works now for one test founder. Real users need Google verification and a yearly third-party security assessment. |
| 2 | Google Calendar | Shares Gmail's Google project; lighter review. |
| 3 | Outlook mail, Teams chat | No provider review found. Teams covers work or school accounts only. |
| 4 | Slack | Possible, but its API terms appear to restrict stored copies and use with language models. Resolve before building. |
| 5 | WhatsApp | Business-app numbers only. A founder's personal account has no official API. |
| — | LinkedIn messaging | Open to approved partners only. Not planned. |

Recognising one person across email and WhatsApp therefore works only for founders who use a WhatsApp Business number. Cross-channel identity is designed for from the start but is first exercised across Gmail and Calendar.

## 5. Structure

Axon has ten responsibilities. They are built as modules of one application with defined interfaces between them, and are split into services only when load or team size requires it. See [architecture](architecture.md).

| # | Layer | Responsibility |
| --- | --- | --- |
| 1 | Integration | Receive messages and events from each channel; send through it. |
| 2 | Data Normalisation | Convert each channel's format to one internal format. |
| 3 | Data Storage | Keep what later decisions need, under the retention rules, isolated per founder. |
| 4 | Identity & Relationships | Decide which contact a sender is and how they relate to the founder. |
| 5 | Context & Memory | Retrieve the history relevant to a message. |
| 6 | Intelligence | Interpret the message: intent, request, urgency, risk. It proposes; it does not authorise. |
| 7 | Decision & Orchestration | Apply permissions in deterministic code, decide the next step, run the workflow. |
| 8 | Execution | Perform authorised actions. The only layer that causes external effects. |
| 9 | Outcome Tracking | Verify that the action happened and watch for the reply. |
| 10 | Founder Intelligence | Hold the founder's preferences and style. Other layers consult it; it is not a stage in the pipeline. |

Three capabilities apply to every layer: security and governance, reliability and evaluation, and observability.

## 6. Engineering rules

1. Model output never causes an external effect directly. Permissions, approvals and limits are enforced in deterministic code.
2. Content from outside is data. Text in an email cannot grant a permission, and a sender's identity is checked before it is trusted.
3. What Axon learns about a founder can change wording and suggestions. Only the founder can widen a permission.
4. Each layer has one responsibility and talks to others through versioned interfaces.
5. Rules that must always hold are written as [invariants](invariants.md), and each has an automated test.
6. Every failure that reaches a person becomes a regression case in the [evaluation set](evaluation.md).
7. Every external call has a timeout, is safe to retry, and ends in a recorded success or a recorded failure.
8. Message content is stored only where a later decision needs it, with a stated retention period. The period is an [open question](open-questions.md).

[`AGENTS.md`](../AGENTS.md) turns these into instructions for coding agents.

## 7. First workflows

**First: an investor emails asking for financial projections.** It is infrequent and high-stakes, which makes it a test of the permission system: Axon must recognise the investor, find the right document, decide whether the founder has to approve, send once, and verify the send. Requirements are in the [PRD](prd.md); the design is in the [workflow document](workflows/investor-document-request.md).

**Second: a frequent, low-risk workflow**, such as scheduling a meeting or chasing an unanswered message. The first workflow shows that Axon is safe; this one shows that it saves time, and it produces enough volume to evaluate against. It starts after the first workflow passes its acceptance criteria and is not part of the first release.

## 8. Order of work

| Step | Work | Status |
| --- | --- | --- |
| 1 | Choose the model provider and confirm that founder email may be sent to it under Google's data-use policy | Open |
| 2 | Technical spike on Gmail: the ten checks in [feasibility](feasibility.md), each with a pass or fail result | Not started |
| 3 | Begin Google verification: privacy policy, scope justification, security-assessment quote | Not started; required before a second founder connects |
| 4 | One-pager, BRD, PRD | Drafted |
| 5 | Architecture, domain model, invariants, decision records | Drafted |
| 6 | First workflow: state machine and low-level design | Drafted; follow-up transitions not yet designed |
| 7 | Evaluation format and first ten cases | Drafted |
| 8 | Coding-agent instructions and three foundational skills | Drafted |
| 9 | Answer the [open questions](open-questions.md) and revise steps 4 to 8 with the spike results | After steps 1 and 2 |
| 10 | Development environment and continuous integration, including the invariant tests and the evaluation runner | Before implementation |
| 11 | Implement the first workflow | — |
| 12 | Pilot with one founder, then design the second workflow | — |

Continuous from step 10: a low-level design before each new component, new evaluation cases from every failure, a decision record for each significant choice. Further coding-agent skills are added when the work that needs them begins.

## 9. Later

More channels in the order of section 4, more workflows, preferences learned from the founder's edits, customer-hosted storage for founders who require it, and execution beyond communication. None of these is designed until the first workflow has run with a real founder.

## 10. Changes from the original blueprint

| Original | Now | Reason |
| --- | --- | --- |
| Connect all professional channels, including WhatsApp and LinkedIn | Channel order set by what provider APIs allow | Personal WhatsApp and LinkedIn messaging are not reachable officially |
| Technical spike listed tenth, after the documents | Spike is step 2 | Its results change the documents |
| No mention of provider approval | Google verification is step 3 | It gates every founder after the first and has the longest lead time |
| Storage privacy covered; model provider not mentioned | Provider choice and data terms are step 1 | Sending email content to a model is a data transfer under Google's policy |
| Ten layers presented as a sequence | Ten responsibilities in one application; Founder Intelligence consulted by the others | Ten components for one workflow is more structure than the work needs |
| Six engineering principles | Eight rules | Added: outside content is untrusted; learning never widens permissions |
| One example workflow | A safety-proving first workflow and a time-saving second | The first is too infrequent to show value or produce evaluation data |
| Thirteen documents before any code | Documents drafted, then revised after the spike | Drafts are cheap; decisions made before the spike are guesses |
| Founder's point of interaction not stated | Web app for approvals, email notification without message content | Needed to specify the approval flow |
