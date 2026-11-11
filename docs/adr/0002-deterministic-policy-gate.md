# ADR 0002: Deterministic policy gate between proposals and commands

- Status: Proposed
- Date: 2026-11-11

## Context

Axon sends communication on a founder's behalf. Model output is probabilistic and message content is untrusted input that may contain instructions. The blueprint requires that permissions, approvals and execution constraints are enforced by deterministic application logic and that the LLM is not the authority on what it may do.

## Decision

1. The Intelligence layer returns a structured interpretation and a draft. Neither contains a tier, a permission or a recipient.
2. Orchestration builds a proposed Action from stored data and the model's output.
3. A policy gate, written as ordinary code with no model call, evaluates the proposed Action against the founder's Permissions and a fixed set of facts. It returns one of three tiers: auto-execute, approval required, blocked.
4. Tier resolution follows INV-7: a never-share Document is blocked; Contact-level Permissions take precedence over relationship-level ones; the most restrictive wins within a level. No matching Permission means approval required.
5. An Action is authorised only by an auto-execute decision or a founder Approval bound to a hash of the Action's content.
6. Execution checks authorisation again immediately before the side effect.

The rules are stated as invariants INV-5 to INV-11 in [`../invariants.md`](../invariants.md).

## Alternatives considered

- Let the model decide whether approval is needed, guided by a prompt. Rejected: not testable to the standard required and open to prompt injection.
- Check policy once, in Orchestration only. Rejected: a Permission can be revoked or content can change between approval and execution.
- A general-purpose policy engine with its own language. Deferred: the MVP's rules are few and a typed function is easier to test.

## Consequences

- Every new action type needs explicit policy facts and tests before it can be automated.
- Behaviour that founders might expect the system to "learn" (for example, loosening approvals) requires an explicit Permission change by the founder.
- Policy decisions are reproducible from recorded facts, which supports audit and regression tests.
