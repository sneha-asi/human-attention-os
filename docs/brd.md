# Axon — Business Requirements Document

The product and the reasoning behind it are described in the [one-pager](one-pager.md). This document covers what the business needs from the product, who it serves, what constrains it, and how success is measured. Product behaviour is specified in the [PRD](prd.md).

## 1. Business objectives

| ID | Objective |
| --- | --- |
| BO-1 | Show that one founder communication workflow can run end to end, with the founder involved only at the approval step or not at all. |
| BO-2 | Reduce the time pilot founders spend on the communication that workflow covers. |
| BO-3 | Earn enough trust that pilot founders widen what Axon may do without approval. |
| BO-4 | Have zero unauthorised actions and zero cross-founder data exposure. |
| BO-5 | Build assets that get more valuable with use and do not depend on one model provider: per-founder context, preferences, permissions, and execution history. |
| BO-6 | Learn which workflows and channels to build next from pilot usage. |

## 2. Target users

### Primary user: the founder

A founder of an operating company who personally handles communication with investors, customers, employees, and partners.

| Characteristic | Consequence for Axon |
| --- | --- |
| High message volume | Triage and routine replies are where time is saved. |
| High-stakes decisions | A wrong send has real cost. Defaults must be conservative. |
| Multiple channels | The same person appears under different identities. |
| Frequent follow-up and coordination | Commitments must be tracked after the reply is sent. |
| Scarce attention | The founder will not maintain a complex tool. Setup and review must be short. |

### Other parties

| Party | Role |
| --- | --- |
| Contacts (investors, customers, employees, co-founders, partners, vendors) | Receive messages sent by Axon on the founder's behalf. They are not users. |
| Delegates (chief of staff, executive assistant) | Possible future users who approve on the founder's behalf. Out of scope for the MVP. |

## 3. Business requirements

| ID | Requirement |
| --- | --- |
| BR-1 | A founder can connect a communication channel and have Axon handle a defined workflow on it. |
| BR-2 | The founder controls what Axon may do without asking, what needs approval, and what is never allowed. |
| BR-3 | Every action Axon takes can be traced to the message that caused it, the decision made, and who or what authorised it. |
| BR-4 | Each founder's data is isolated from every other founder's. |
| BR-5 | Axon stores only what it needs to function, and the founder can have it deleted. |
| BR-6 | Axon's behaviour is measured against a test set, and each observed failure is added to that set. |
| BR-7 | The product does not depend on one model provider. |
| BR-8 | The first release proves one workflow completely before a second is started. |

## 4. Scope

### In scope for the first release

- One founder per account.
- Gmail as the only channel.
- One workflow: an investor requests a financial document. See the [PRD](prd.md#2-mvp-scope).
- A pilot with a small number of founders, using synthetic scenarios and consented real data.

### Out of scope for the first release

- WhatsApp, Slack, Microsoft Teams, LinkedIn, and calendar integrations.
- Workflows other than the investor document request.
- Execution automation beyond communication.
- Customer-hosted deployment.
- Team or delegate accounts.
- Learning founder preferences from behaviour without the founder confirming them.

## 5. Constraints

| Area | Constraint |
| --- | --- |
| Platform access | Channel APIs set what is possible. Gmail access to mail content requires Google's OAuth verification for restricted scopes. WhatsApp and LinkedIn have limited or no API access for personal accounts. Details are in [feasibility](feasibility.md). |
| Privacy | Founder mail contains third parties' information and confidential company data. Storage is minimised, encrypted in transit and at rest, and isolated per founder. |
| Consent | Real founder data is used in the pilot and in evaluation only with the founder's consent. |
| Authority | The model proposes. Deterministic code decides what is permitted. This rules out designs where the model's output triggers a send directly. |
| Model dependence | Model providers are used through an interface so they can be replaced. |
| Stakes | Messages go to investors and customers under the founder's name. An error is visible outside the company and may not be reversible. |
| Team and time | Not yet defined. See open questions. |

## 6. Business success metrics

The blueprint sets no numeric targets. Targets are open questions and should be set after the pilot baseline is measured.

| ID | Metric | How it is measured in the pilot | Target |
| --- | --- | --- | --- |
| M-1 | Unauthorised actions | Count of executed actions with no matching permission or approval record, from the audit log. Reviewed weekly. | 0 |
| M-2 | Cross-founder data exposure incidents | Count from access logs and incident reports. | 0 |
| M-3 | Workflow completion rate | In-scope requests that reach a verified outcome, divided by in-scope requests received. | Open |
| M-4 | Founder time per request | Baseline: the founder's self-reported time to handle a comparable request, collected in a pre-pilot interview. Pilot: time from the approval prompt being opened to the decision, from application logs. | Open |
| M-5 | Approval without edits | Approved drafts sent unchanged, divided by all approved drafts. | Open |
| M-6 | Rejection rate | Proposals the founder rejected, divided by proposals shown. | Open |
| M-7 | Time to response | Median time from the investor's email arriving to the reply being sent, compared with the same founder's median for comparable threads in the three months before the pilot. | Open |
| M-8 | Autonomy expansion | Number of permissions each founder moves from approval required to auto-execute during the pilot. | Open |
| M-9 | Evaluation pass rate | Share of cases in the evaluation set that pass on each release. See [evaluation](evaluation.md). | Open |
| M-10 | Pilot retention | Founders with the channel still connected at the end of the pilot, divided by founders who started. | Open |

M-1 and M-2 are release gates. A breach of either stops the pilot until the cause is found and covered by a test.

## 7. Expected outcomes

At the end of the first pilot the business should have:

- a working implementation of one workflow, with measured values for M-1 to M-10;
- an evaluation set that includes every failure seen during the pilot;
- evidence from pilot founders about which workflow and channel to build next;
- a decision on whether to continue to a second workflow.

## 8. Assumptions and open questions

### Assumptions made in this document

- The buyer and the user are the same person, the founder.
- The pilot is free. Pricing is not addressed.
- "Small number of founders" for the pilot is left unquantified. The metrics work for any number from one upward, but M-10 is not meaningful below about five.
- M-4 relies on self-reported baselines because there is no instrumentation before Axon is connected.

### Open questions

1. What are the numeric targets for M-3 to M-10?
2. How many founders are in the pilot, how long does it run, and how are they recruited?
3. What stage and size of company is the target founder? A pre-seed founder and a Series B founder have different message volumes and different investors.
4. What is the business model: subscription per founder, usage-based, or something else?
5. Which regions will pilot founders be in? This decides which data-protection rules apply.
6. What team, budget, and timeline are available for the first release?
7. Which workflow comes second? Candidates from the blueprint are follow-ups, scheduling, and delegation.
