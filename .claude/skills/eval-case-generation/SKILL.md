---
name: eval-case-generation
description: Write evaluation cases for Axon in the format defined in docs/evaluation.md. Use when a bug or wrong behaviour is found, when a workflow, state, permission rule or requirement is added or changed, or when asked for test scenarios or regression cases.
---

# Eval case generation

A case fixes the inputs and states exactly what Axon must do with them. One case tests one behaviour.

## Read first

1. `docs/evaluation.md`: the schema, the allowed values and the existing cases.
2. The workflow document under `docs/workflows/` for the states and decisions the case will assert on.
3. `docs/invariants.md` and `docs/prd.md` for the INV-n and FR-n the case covers.
4. `evals/cases/`: check the behaviour is not already covered.

## Where the output goes

One file per case: `evals/cases/<workflow>/<id>.yaml`, where `<id>` is the next free number for that workflow (for example `IDR-011`). Add a row to the case index in `docs/evaluation.md`.

## Procedure

1. **Name the behaviour** in one sentence: given what, Axon must do what.
2. **Build the smallest fixture that triggers it.** Only the Contacts, Permissions, documents, prior Messages and incoming Messages the behaviour needs. Synthetic data only, with `example.com` addresses.
3. **Write the expectations.** Fill every `expect` field. Use exact values for identity, decision, autonomy tier, action and final state. Put free-text requirements in `reply_must_include` and `reply_must_not_include`; do not assert on full reply wording.
4. **Write what must not happen** in `must_not`. For a case about permissions, injection or duplicates, this is the main assertion.
5. **Link it.** Fill `covers` with the FR-n and INV-n IDs. For a regression, fill `origin` with the date and a one-line description of the failure.
6. **Add neighbours.** For each new case, consider the opposite outcome and one boundary: the same input with the Permission present and absent, a known and an unknown sender, a first and a repeated delivery. Write the ones not already covered.
7. **Run it.** A regression case must fail on the code before the fix and pass after. Say in the change description that you checked both.

## Checklist

- [ ] One behaviour per case; the title states it.
- [ ] Every `expect` field is filled, including `must_not`.
- [ ] Decision, tier, action and state values come from the lists in `docs/evaluation.md`.
- [ ] No real names, addresses, companies or documents.
- [ ] The case does not depend on another case having run.
- [ ] A fault needed by the case is declared under `given.faults`, not assumed.
- [ ] `covers` lists at least one FR-n or INV-n.
- [ ] The case index in `docs/evaluation.md` is updated.
- [ ] An existing case's expected values were not changed to make a run pass.
