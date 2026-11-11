# Axon — Open Questions

Decisions the documents could not take from the blueprint. Each has the default the documents currently assume, and the documents that change if the answer is different. Answer in any order; unanswered questions keep their default.

## Product behaviour

| # | Question | Current default | Changes if different |
| --- | --- | --- | --- |
| 1 | When a Contact-level Permission and a relationship-level Permission disagree, which wins? | The Contact-level one, even when it is looser (for example "investors: blocked" with "Ravi: auto-execute" sends to Ravi). | `prd.md` FR-15, `invariants.md` INV-7, workflow section 4 |
| 2 | Should a pending approval expire? | Yes. Reminders at 24 and 72 hours, expiry at 7 days, nothing sent. | `prd.md` FR-40, workflow T10 |
| 3 | Should the investor get a holding reply while a request waits for the founder? | No. Nothing is sent until approval. | `prd.md`, workflow, `evaluation.md` |
| 4 | Should auto-sent replies say they were sent by an assistant, or be delayed so they do not look automated? | No disclosure, no added delay. | `prd.md` NFR-5 and drafting requirements |
| 5 | May the founder approve a reply to a known Contact whose message failed sender authentication? | Yes, with a warning on the approval item. | `invariants.md` INV-8, workflow section 4 |
| 6 | Should the reply go to everyone on the thread when the investor copied colleagues? | No. Only to the sender. | Workflow assumption 6, `invariants.md` INV-10 |
| 7 | If the same Document version was already sent to this Contact, should a repeat request be treated differently? | No. The founder is shown that it was sent before. | Workflow section 4 |
| 8 | After how many days without a reply should Axon propose a follow-up? | Configurable; no value set. | `prd.md` FR-33 |
| 9 | Should the founder be able to approve from the notification email or a phone app? | No. Web app only; the email is a content-free alert. | `prd.md` FR-24, `architecture.md` |
| 10 | Should Axon suggest Contacts and categories from the inbox during setup? | No. The founder enters them. | `prd.md` FR-5, `domain-model.md` |
| 11 | Which document types beyond financial projections belong in the first release? | Any Document the founder registers, as long as the request is for a financial document. | `prd.md` section 2 |
| 12 | Is a 15-minute bounce window acceptable before a request is shown as completed? | Yes. | Workflow T15 |
| 13 | How many send retries before the founder is told? | 5 attempts over about 30 minutes for retryable errors; none for rejected sends. | Workflow section 10, `prd.md` FR-31 |

## Design gap

| # | Question | Current default | Changes if different |
| --- | --- | --- | --- |
| 14 | Follow-up proposals (FR-33) are required by the PRD but only outlined in the workflow document. Keep them in the first release, or defer them? | Kept. The workflow document must gain their states and transitions before implementation. | `prd.md` FR-33 and AC-15, workflow section 12, `architecture.md` Layer 9 |

## Data and privacy

| # | Question | Current default | Changes if different |
| --- | --- | --- | --- |
| 15 | Which model provider, and may message content be sent to a third-party model API? Google's Limited Use policy on this is unconfirmed ([feasibility](feasibility.md)). | Undecided. A provider interface keeps the choice open; NFR-12 requires terms that prohibit training. | `adr/0001-stack.md`, `prd.md` NFR-12 |
| 16 | How long are message bodies and raw items kept while the account is active, and how quickly is data removed after account deletion? | Configurable; no values set. | `prd.md` FR-38, `architecture.md` Layer 3 |
| 17 | After a founder deletes their account, are content-free audit records kept? | Either is allowed. | `invariants.md` INV-4 and INV-17 |
| 18 | Do message bodies need field-level encryption in addition to encrypted volumes? | No. | `architecture.md` section 5A |
| 19 | Where is the pilot hosted? | Undecided. Local disk and Docker Compose only so far. | `architecture.md` section 6 |

## Domain model

| # | Question | Current default | Changes if different |
| --- | --- | --- | --- |
| 20 | Are relationship categories a fixed list or founder-defined? | Fixed list. | `domain-model.md` Contact |
| 21 | Can a Contact hold more than one relationship, such as an investor who is also a customer? | No. One relationship. | `domain-model.md`, policy resolution |
| 22 | Can a Permission target a group of Contacts, such as one fund? | No. One Contact or a whole category. | `domain-model.md` Permission, `prd.md` FR-14 |

## Evaluation

| # | Question | Current default | Changes if different |
| --- | --- | --- | --- |
| 23 | How is reply wording scored beyond the include and exclude lists? | Not scored. | `evaluation.md` Scoring |

## Business and pilot

| # | Question | Current default | Changes if different |
| --- | --- | --- | --- |
| 24 | What are the numeric targets for metrics M-3 to M-10? | None. Set after the pilot baseline. | `brd.md` section 6 |
| 25 | How many founders are in the pilot, for how long, and how are they recruited? | "A small number"; unquantified. | `brd.md` |
| 26 | What stage and size of company is the target founder, and in which regions? Regions decide which data-protection rules apply. | Undecided. | `brd.md` section 2 |
| 27 | What is the business model? | The pilot is free; pricing is not addressed. | `brd.md` |
| 28 | What team, budget and timeline are available for the first release? | Undecided. | `brd.md` section 5 |
| 29 | Which workflow comes second? | Undecided. Candidates in the blueprint: follow-ups, scheduling, delegation. | `brd.md`, `feasibility.md` channel order |
