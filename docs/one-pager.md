# Axon — One-Pager

## What it is

Axon is a system that manages a founder's professional communication. It reads incoming messages, works out what each one asks for, and replies, follows up, or schedules within permissions the founder has set. It asks the founder only when a decision is theirs to make.

## The problem

A founder's working day is split between communication and execution, and communication takes a large share of it. Investors, customers, employees, and partners write on several channels. Each message needs the founder to recall the history, decide what to do, write a reply, and remember to follow up. Most of these steps repeat. A few carry real consequences, and they arrive mixed in with the rest.

Current AI tools help with single steps: drafting a reply, summarising a thread. The founder still reads every message, decides, sends, and tracks the result.

## The insight

AI capability is becoming abundant. Human attention is fixed. The useful question for each message is whether it needs the founder at all.

| | Who understands the work | Who executes | Where the human is needed |
| --- | --- | --- | --- |
| Before | Human | Human | Everywhere |
| Today | Human | AI, when asked | To read, delegate, and check |
| Axon | AI | AI, within permissions | Where their judgment is required |

## Who it is for

Founders first, for five reasons:

- they handle a high volume of professional communication;
- much of it involves high-stakes decisions;
- it is spread across multiple channels;
- it needs frequent follow-up, delegation, and coordination;
- their attention is expensive.

## How it works

An investor emails a founder asking for the latest financial projections.

1. Axon receives the email and recognises the sender as an existing investor.
2. It retrieves the history: the founder promised updated projections after the last call.
3. It classifies the message as a request for a financial document.
4. It checks the founder's permissions. Sharing financials with this investor needs approval.
5. It drafts a reply in the founder's style, attaches the document, and asks the founder to approve.
6. The founder approves. Axon sends the email.
7. Axon confirms the email was sent and keeps the conversation open until the investor responds.

If the founder had pre-authorised this investor for this document, steps 5 and 6 would happen without them. If the permissions blocked it, nothing would be sent.

The model interprets messages and proposes actions. Permissions and approvals are enforced by ordinary application code, so the model cannot authorise itself.

## Where it starts and where it goes

The first version covers one workflow for one founder on Gmail: the investor document request above. See the [PRD](prd.md) for scope and the [architecture](architecture.md) for how it is built.

Later versions add channels (WhatsApp, Slack, Teams, LinkedIn, calendars), more workflows, and a deeper model of how each founder decides. The long-term direction is an operating layer that handles execution as well as communication. That expansion depends on the first workflow proving reliable.

## Related documents

- [BRD](brd.md): business objectives, constraints, metrics
- [PRD](prd.md): requirements, MVP scope, acceptance criteria
- [Architecture](architecture.md): the ten layers and cross-cutting capabilities
