# ADR 0005: Send Gmail replies as draft-then-send

- Status: Proposed, pending the feasibility spike
- Date: 2026-11-11

## Context

Invariant INV-13 requires at most one sent message per Action. A send call can time out after Gmail has accepted it, leaving the outcome unknown. Gmail's send endpoint takes no client-supplied idempotency key, so a blind retry risks a duplicate email to an investor.

## Decision

Execution sends a reply in two recorded steps:

1. `users.drafts.create` with the full message. Store the draft ID on the Action.
2. `users.drafts.send`. Store the returned message ID on the Action.

If step 2 has an unknown outcome, the connector reconciles before any retry: a draft that still exists was not sent; a draft that is gone together with a `SENT` message in the thread matching recipient and attachment hash was sent. If neither can be established, the Workflow is escalated and Axon does not resend.

This requires the `gmail.compose` scope.

## Alternatives considered

- `users.messages.send` with retry. Rejected: no way to tell a lost response from a failed send.
- `users.messages.send` with a self-generated `Message-ID` header and a search by that ID. Not chosen: depends on Gmail preserving the header, which needs verification.

## Consequences

- A draft is briefly visible in the founder's Drafts folder.
- Two API calls per send.
- The reconciliation logic depends on Gmail removing a draft when it is sent and on thread reads being consistent shortly after. The feasibility spike ([`../feasibility.md`](../feasibility.md)) must confirm both; if either fails, this ADR is revisited.
