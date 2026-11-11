# Pre-Coding Readiness Plan

What has to be true before implementation starts, what can wait, and where this repository stands against that bar today.

The planning structure is roughly 80–85% complete. No further set of documents is missing. What is missing is **validation, feasibility, MVP boundaries, and one proven end-to-end workflow**.

> Document the architecture before implementing it, but let engineering discoveries keep improving the architecture.

Avoid both extremes: coding without a system design, and spending months designing a system that has not been tested in reality.

## 1. Sequence

1. Idea and one-pager.
2. BRD.
3. PRD.
4. System architecture: the ten layers and the cross-cutting capabilities.
5. System design: boundaries, invariants, and contracts.
6. LLD, written per component, just before that component is implemented.
7. Technical validation: integration feasibility and the technical spike.
8. First end-to-end workflow selected and designed.
9. Coding-agent setup: `AGENTS.md`, other instruction files, and the foundation skills.
10. Development environment and CI.
11. Build the first vertical slice.

Documentation and code then grow together. Deeper LLDs and additional skills are added alongside the work that needs them.

## 2. Gaps to close before coding

| Gap | Question to answer | Why it matters |
| --- | --- | --- |
| Problem validation | Have target users confirmed this is a painful problem? | Prevents building something nobody needs. |
| MVP definition | What is the smallest complete version of the product? | Prevents building all ten layers at once. |
| Technical feasibility | Can the required platform APIs be accessed, and can actions be executed through them? | Some integrations carry significant restrictions. |
| Data strategy | What data is collected, retained, retrieved, and deleted? | The intelligence layer depends on it. |
| Permission architecture | What may the system do on its own, and what needs approval? | Prevents unauthorised actions. |
| Acceptance criteria | How do we know a feature works? | Gives developers and coding agents an objective finish line. |
| Cost and infrastructure | What do model calls, storage, integrations, and execution cost? | Prevents an architecture that is too expensive to run. |
| Deployment strategy | How do we deploy, monitor, roll back, and recover? | Prevents fragile releases. |

Three of these carry the most weight.

### 2.1 Problem validation

The architecture should reflect observed behaviour, not assumptions. Find out from target users:

- Which communication consumes most of their time?
- Which tasks do they repeatedly postpone?
- What do they already delegate?
- What would they trust AI to execute independently?
- What would they never let AI do without approval?

### 2.2 Technical feasibility

Do not design around an integration until it is known what the integration permits. Investigate:

- Gmail API capabilities and limits.
- WhatsApp Business API capabilities and limits.
- Slack API permissions.
- Calendar integrations.
- OAuth and account authentication.
- Webhooks, rate limits, and API restrictions.
- Whether each required action can actually be automated.

### 2.3 First complete workflow

Pick one workflow that cuts through every layer and build it end to end before adding capabilities. Illustrative example, an investor email arriving for a founder:

1. Receive the email.
2. Identify the investor.
3. Retrieve the relevant conversation history.
4. Understand what is being asked.
5. Determine the required action.
6. Prepare a response.
7. Request approval if necessary.
8. Send the response.
9. Verify that it was sent.
10. Track whether the conversation needs further action.

## 3. Technical spike

Before committing to the full architecture, build a small proof of concept. It is not the product. It tests the hardest assumptions:

- Can communication be ingested reliably?
- Can the correct context be retrieved?
- Can an LLM produce a structured action proposal?
- Can permissions be enforced independently of the LLM?
- Can an action be executed and its outcome verified?
- Can the system recover safely when an operation fails?

## 4. Checklist and current status

Status is assessed against the files in this repository.

| # | Requirement | Target | Current status |
| --- | --- | --- | --- |
| 1 | One-pager | Clear | Done: [`VISION.md`](VISION.md), [`README.md`](README.md). |
| 2 | BRD | Defined | Missing. |
| 3 | PRD | Initial version complete | Done: [`PRD.md`](PRD.md). |
| 4 | Ten-layer architecture | Defined | Missing as such. [`LLD.md`](LLD.md) §4 defines nine service boundaries, not ten layers. |
| 5 | Cross-cutting capabilities | Defined | Missing as a named list. Pieces appear in [`LLD.md`](LLD.md) §3 and §8. |
| 6 | Domain model | Initial version complete | Partial: [`LLD.md`](LLD.md) §4.3 and §5, [`BUILD_SPEC.md`](BUILD_SPEC.md) minimum data objects. |
| 7 | System invariants | Defined | Partial: stated across [`LLD.md`](LLD.md) §3 and §8, not consolidated. |
| 8 | Workflow state machine | First workflow defined | Done: action state machine in [`LLD.md`](LLD.md) §5. |
| 9 | Technical feasibility research | First integrations validated | Not started. [`BUILD_SPEC.md`](BUILD_SPEC.md) assumes mock connectors only. |
| 10 | MVP scope | Clearly restricted | Done: [`PRD.md`](PRD.md) §7, §12, §13 and [`BUILD_SPEC.md`](BUILD_SPEC.md). |
| 11 | Acceptance criteria | Defined for first workflow | Partial: [`BUILD_SPEC.md`](BUILD_SPEC.md) definition of done. Not yet testable per step. |
| 12 | Security and permissions | Initial model defined | Done at initial level: [`PRD.md`](PRD.md) §10, [`BUILD_SPEC.md`](BUILD_SPEC.md) policy defaults. |
| 13 | Coding-agent instructions | Ready | Missing. No `AGENTS.md` or skills. |
| 14 | Development environment and CI | Set up | Missing. |

Also open from section 2, with no document yet: problem validation (planned as Phase 0 in [`PRD.md`](PRD.md) §14, not yet run), data strategy, cost and infrastructure, deployment strategy.

## 5. Coding-agent setup

Instruction files (`AGENTS.md` and related) and skills are written once the first MVP workflow design is locked.

Skills to prioritise. Only the foundations are needed at the start:

- Architecture planning
- Impact analysis
- System design
- LLD generation
- Database design
- API and integration development
- Workflow engineering
- Testing and evaluation
- Security review
- Code review

## 6. Deferred until needed

Do not complete these before coding. Write each when the work that needs it begins.

- Every individual LLD.
- Every possible API contract.
- Every coding-agent skill.
- Every future workflow.
- A complete microservices architecture.
- All possible AI evaluation scenarios.
- Scaling infrastructure for millions of users.

For example, the identity-resolution skill is written when identity resolution is being implemented.

## 7. Open decisions

The planning discussion this document records and the existing documents disagree on three points. They need a decision before the BRD is written.

1. **Name.** The discussion calls the product *Axon*. The repository calls it *Human Attention OS*.
2. **Initial user.** The discussion targets founders delegating their communication lifecycle. [`PRD.md`](PRD.md) §6 targets Sarvam Strategy & Operations.
3. **First workflow.** The discussion uses an inbound investor email as its example. [`PRD.md`](PRD.md) §6 and [`BUILD_SPEC.md`](BUILD_SPEC.md) specify cross-functional initiative follow-through from voice or text input.

The integrations to validate in section 2.2 depend on the answers: the founder framing needs Gmail and WhatsApp first, while the current build spec needs only Sarvam APIs and mock connectors.
