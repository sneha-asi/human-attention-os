---
name: architecture-impact-analysis
description: Work out what a proposed change affects before making it. Use when a change touches a layer interface, an API or event contract, a domain entity, a workflow state, an invariant, a permission rule or a dependency, or when asked whether a change is safe.
---

# Architecture impact analysis

Do this before editing. The result is a short written analysis, then a go, a go with conditions, or a stop.

## Procedure

1. **State the change in one sentence.** Name the thing being changed: an interface, entity, state, invariant, permission rule or dependency.
2. **Find the owner.** Use the table in `AGENTS.md` to find the document that owns it. Read that section.
3. **Find every user of it.** Search the docs and the code for its name:
   - `docs/architecture.md` for layers that call the interface;
   - `docs/domain-model.md` for entities that reference the entity;
   - `docs/workflows/` and `docs/lld/` for states and steps that depend on it;
   - `docs/invariants.md` for INV-n that mention it;
   - `evals/cases/` and the tests for cases that assert on it.
4. **Classify each contract change.** Adding an optional field is compatible. Removing or renaming a field, changing a type, changing meaning, or adding a required field is breaking and needs a new version.
5. **Check each invariant.** For every INV-n the change could affect, write how it still holds, naming the check that enforces it. If you cannot, the change stops.
6. **Check the control boundary.** Confirm the change does not move a permission, approval or execution decision into a prompt or model output, and does not let one layer reach into another's storage.
7. **Check failure behaviour.** Confirm retries, duplicates and timeouts on the changed path still cannot cause a second send or a false success.
8. **List the follow-on work.** Documents to update, tests and eval cases to add or change, migrations, and whether an ADR is needed (it is for a new dependency, a new layer interaction, or reversing an earlier ADR).

## Output

Put the analysis in the change description. If the change needs an ADR, put it there as well. Use this shape:

```text
Change:
Owner document:
Affected: <layers, entities, states, contracts, eval cases>
Contract changes: <compatible | breaking → new version>
Invariants: <INV-n: how it still holds>
Follow-on work:
Verdict: <go | go with conditions | stop>
```

## Checklist

- [ ] Every user of the changed thing is listed, found by search and not from memory.
- [ ] Every breaking contract change has a new version.
- [ ] Every affected invariant has a named enforcing check and a test.
- [ ] No permission or approval logic moved out of deterministic code.
- [ ] The owning document is updated in the same change.
- [ ] A verdict of stop is reported to the user with the reason; do not work around it.
