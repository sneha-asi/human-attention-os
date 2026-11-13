# Axon

Axon manages a founder's professional communication. It reads incoming messages, works out what each one asks for, and replies within permissions the founder has set, asking the founder only when the decision is theirs. The first release covers one workflow for one founder on Gmail: an investor asks for financial projections, and Axon sends the document with or without the founder's approval, then verifies the send.

**Status:** design documents only. No code has been written yet.

## Documents

Read in this order.

| Document | Contents |
| --- | --- |
| [`docs/blueprint.md`](docs/blueprint.md) | Top-level plan: structure, channel order, engineering rules, order of work |
| [`docs/blueprint-original.md`](docs/blueprint-original.md) | The original blueprint, word for word, kept as the source |
| [`docs/one-pager.md`](docs/one-pager.md) | What Axon is, the problem, why founders first |
| [`docs/brd.md`](docs/brd.md) | Business objectives, constraints, and how success is measured |
| [`docs/prd.md`](docs/prd.md) | Requirements, MVP scope, acceptance criteria |
| [`docs/feasibility.md`](docs/feasibility.md) | What each channel's API allows, risks, and the spike plan |
| [`docs/architecture.md`](docs/architecture.md) | The ten layers, which may call which, cross-cutting mechanisms, stack |
| [`docs/domain-model.md`](docs/domain-model.md) | Entities and how they relate |
| [`docs/data-strategy.md`](docs/data-strategy.md) | What is stored, retrieved, sent out and deleted |
| [`docs/invariants.md`](docs/invariants.md) | Rules that must always hold, with how each is enforced and tested |
| [`docs/workflows/investor-document-request.md`](docs/workflows/investor-document-request.md) | State machine and low-level design of the first workflow |
| [`docs/adr/`](docs/adr/) | Architecture decisions and the alternatives considered |
| [`docs/evaluation.md`](docs/evaluation.md) | Test-case format and the first ten cases |
| [`AGENTS.md`](AGENTS.md) | Rules for coding agents; skills are in `.claude/skills/` |

## Decisions still needed

[`docs/open-questions.md`](docs/open-questions.md) lists every undecided question with the default the documents assume.

## Earlier work

The Human Attention OS documents that preceded Axon are in [`archive/human-attention-os/`](archive/human-attention-os/). They are superseded.
