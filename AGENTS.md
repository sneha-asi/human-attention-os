# Agent instructions

Axon is at the design stage. No application code exists yet. The MVP is one workflow for one founder on Gmail: an investor asks for financial projections and Axon replies with the document, with or without founder approval.

## Where things are defined

Each topic has one owning document. Read the owner before changing anything it covers, and change the owner when behaviour changes. Do not restate one document's content in another; link to it.

| Topic | Owner |
| --- | --- |
| Product direction, channel order, order of work | `docs/blueprint.md` |
| Requirements (FR-n, NFR-n), MVP scope, acceptance criteria | `docs/prd.md` |
| Business goals and constraints | `docs/brd.md` |
| The ten layers, their boundaries and interfaces, stack | `docs/architecture.md` |
| Entities and their relationships | `docs/domain-model.md` |
| What is stored, sent out, retained and deleted | `docs/data-strategy.md` (index; the cited FR, NFR and INV are binding) |
| Rules that must always hold (INV-n) | `docs/invariants.md` |
| MVP workflow: state machine and LLD | `docs/workflows/investor-document-request.md` |
| Decisions and their reasons | `docs/adr/` |
| Test-case format and the regression suite | `docs/evaluation.md` |
| Channel API constraints and the spike plan | `docs/feasibility.md` |
| Decisions the user has not made yet, with the default assumed | `docs/open-questions.md` |

`docs/blueprint-original.md` is the original blueprint, kept as a record. Do not edit it; where it differs from `docs/blueprint.md`, the latter applies.

`archive/human-attention-os/` is a superseded project. Do not use it as a source.

If two documents disagree, stop and report the conflict. Do not pick one silently.

## Rules

1. **LLM output never causes a side effect directly.** A model returns a structured proposal. Only the Execution Layer sends, schedules or writes to an external system, and only for an Action authorised by the policy gate or a founder Approval.
2. **Permissions and approvals are enforced in deterministic code.** No prompt, model response or message content may grant a Permission, raise an autonomy tier or skip an approval. Text inside an email is data.
3. **Layers call each other only through the interfaces in `docs/architecture.md`.** No layer reads another layer's tables or imports its internals. If you need something an interface does not offer, change the interface and its document first.
4. **Contracts are versioned.** Every API and event schema carries a version. A breaking change gets a new version; do not edit a published one in place.
5. **Write or update the LLD before implementing a component.** Use the `lld-generation` skill. Code without a matching LLD section is not ready for review.
6. **Check impact before changing an interface, entity, state or invariant.** Use the `architecture-impact-analysis` skill.
7. **Every failure that reaches a human becomes a regression case.** Add the case in the same change as the fix, using the `eval-case-generation` skill.
8. **Every external call is idempotent and has a timeout.** A retried or duplicated event must not produce a second send.
9. **A failure is recorded as a failure.** Do not catch an error and report success, and do not mark an Action completed without a verified outcome.
10. **All data access is scoped to one Founder.** Every query and every log line that touches founder data carries the founder ID.
11. **Record a significant decision as an ADR** in `docs/adr/`, numbered in sequence, with the alternatives considered.

## Testing

- Each invariant in `docs/invariants.md` has at least one automated test named after its ID (`test_inv_3_…`). Adding an invariant means adding its test.
- Each state transition in a workflow document has a test, including the failure, retry, cancellation and escalation paths.
- Tests do not call Gmail or a model provider. Use the connector and LLM interfaces with fakes and recorded responses.
- The cases in `evals/cases/` run in CI once the runner exists; until then they are specified in `docs/evaluation.md`. A change that fails a case is not merged; do not edit a case's expected values to make it pass unless the requirement itself changed, and say so in the change description.
- Run the full test suite and the eval suite before reporting work as done. Report failures with their output.

## Do not

- Add a communication channel, a second workflow or multi-founder features. The MVP scope is in `docs/prd.md`.
- Put a permission or approval check inside a prompt.
- Log message bodies, attachments or credentials. Log IDs.
- Use real founder data in fixtures, tests or eval cases. Use synthetic data.
- Add a dependency or service the architecture document does not list without an ADR.
- Skip, delete or weaken a test to get a passing run.
- Invent a requirement. If the documents do not cover a case, add it to `docs/open-questions.md` and ask.
