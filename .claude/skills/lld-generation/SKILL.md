---
name: lld-generation
description: Write or update the low-level design for an Axon component or workflow. Use before implementing a new component, before changing an existing component's schema, API, events or state transitions, or when asked for an LLD.
---

# LLD generation

Produce a design specific enough that someone could implement the component from it without asking questions.

## Read first

1. `docs/architecture.md`: the layer the component belongs to and the interfaces it may use.
2. `docs/domain-model.md`: the entities it reads or writes.
3. `docs/invariants.md`: every INV-n the component can affect.
4. `docs/prd.md`: the FR-n and NFR-n it implements.
5. `docs/workflows/investor-document-request.md`: the existing LLD, for format and for states you must not contradict.
6. `docs/adr/`: decisions that constrain the design.

## Where the output goes

- A workflow: `docs/workflows/<workflow-name>.md`.
- A component that serves several workflows: `docs/lld/<component-name>.md`.
- Updating an existing design: edit its file. Do not create a second one.

## Sections to write

1. **Scope.** What the component does, which layer owns it, which FR-n it covers, what it does not do.
2. **Interfaces.** Each function, endpoint or event it exposes or consumes: name, version, input and output fields with types, error cases.
3. **Data.** Tables or stores with columns, types, keys, indexes and the founder-scoping column. Retention for each.
4. **Logic.** Steps in order. Mark which steps use a model and which are deterministic. Every permission or approval check must be a deterministic step.
5. **States.** A table of state, event, next state and guard. Include failure, retry, timeout, cancellation and escalation.
6. **Failure handling.** For each external call: timeout, retry policy, idempotency key, what is recorded, when a human is told.
7. **Invariants.** Each relevant INV-n and the step or check that upholds it.
8. **Tests.** The unit tests and the eval cases that must exist before this is done.
9. **Assumptions.** Put questions for the user in `docs/open-questions.md`, with the default the design assumes.

## Checklist

- [ ] Every field has a type, and every interface has a version.
- [ ] No step lets model output trigger a send, write or schedule without passing the decision and permission checks.
- [ ] Every state has a way out; no transition is missing for failure or timeout.
- [ ] Every external call has a timeout and an idempotency key.
- [ ] Every table is scoped to a Founder.
- [ ] Terms match `docs/domain-model.md` and layer names match `docs/architecture.md`.
- [ ] Nothing restates another document; it links instead.
- [ ] If the design changes an interface, entity, state or invariant that already exists, the `architecture-impact-analysis` skill has been run.
- [ ] Choices you made without a source are listed under assumptions.
