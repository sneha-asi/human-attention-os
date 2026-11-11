# Human Attention OS — PRD

## 1. Principle

As AI becomes capable of handling more communication and execution, human attention becomes the scarce resource. People should spend it on work that requires judgment, and AI should handle the rest.

Human Attention OS is an execution layer that sits above existing communication and work systems. It handles predictable communication and follow-through, and involves people only when their judgment, creativity, empathy, accountability, or relationships are needed.

## 2. Why Sarvam

Indian organizations are multilingual, voice-heavy, code-mixed, and spread across many channels. English-first meeting assistants and dashboard-driven workflows do not fit them.

Sarvam can make execution independent of language: people communicate however they naturally do, and the system converts that into a common set of executable actions.

```text
Any language / voice / channel
            ↓
Intent and context understood
            ↓
Common executable action
            ↓
AI coordinates and follows through
            ↓
Human attention only when necessary
```

The product should dogfood Sarvam's own models and Voice Agent capabilities internally, then serve as a reference architecture for multilingual enterprise operations.

## 3. The problem

People spend too much time on work around work:

- writing routine messages;
- repeating context;
- coordinating across teams;
- finding owners;
- scheduling and rescheduling;
- chasing updates and asking for status;
- copying information between tools;
- escalating overdue work;
- deciding which notifications deserve attention.

Existing AI tools summarize communication. They do not carry predictable work through to completion, and they do not decide whether an item deserves human attention in the first place.

## 4. The Attention Firewall

The system decides whether work should reach a human at all, before it reaches them.

Every incoming request, message, meeting invite, escalation, or task is evaluated against context, urgency, value, risk, and the person's role. The system can then:

- resolve it automatically;
- delegate it;
- batch it;
- defer it;
- decline or remove it;
- ask one focused clarification question;
- surface it to the right human with the relevant context.

## 5. Initial users and first workflow

Start with **Sarvam Strategy & Operations**, then validate with Partnerships and Sales. These teams coordinate across functions, manage many follow-ups, work across multiple tools, and can measure operational improvement.

The first workflow is **cross-functional initiative follow-through**, which begins where meeting AI usually stops:

```text
Conversation
→ commitment identified
→ owner and deadline assigned
→ follow-up sent
→ progress checked
→ completion evidence verified
→ delay escalated
→ action closed
```

The target outcome: routine communication happens without human involvement, and overdue and blocked work decreases.

## 6. MVP scope

The MVP is a simple internal web application with voice input and spoken summaries. It accepts approved inputs from meeting notes, transcripts, recordings, email threads, and Slack or internal chat messages.

It then:

1. Identifies commitments, decisions, requests, owners, deadlines, dependencies, and uncertainty.
2. Converts commitments into structured actions.
3. Uses company and personal context to propose the correct owner and priority.
4. Drafts or sends low-risk internal follow-ups.
5. Creates tasks in one connected task system.
6. Requests status updates at context-aware times.
7. Verifies completion through explicit confirmation and system evidence.
8. Escalates blocked or overdue work.
9. Provides a daily human-attention briefing.
10. Records an audit history and learns from personal and organizational feedback.

## 7. Interaction model

- **Voice:** delegate work, give updates, ask questions, and report completion.
- **Text:** correct, clarify, and instruct the system.
- **Web:** review actions, context, permissions, and exceptions.
- **Background automation:** handle approved routine work.
- **Exception inbox:** show decisions, risks, ambiguity, blocked work, and high-value human actions.

The primary view is the exception inbox: what needs the person, rather than a feed of everything that happened.

## 8. Context and learning

The system needs a permissioned context layer containing people and roles, teams and reporting relationships, projects and priorities, responsibilities and ownership patterns, policies and approval rules, communication preferences, deadlines and dependencies, connected systems and available actions, and relevant documents and approved conversation history.

It learns at two levels:

- **Personal context:** what an individual prefers to automate, review, delay, delegate, or escalate.
- **Organizational context:** how Sarvam works, who owns what, which workflows exist, and what policies apply.

Learning signals include explicit approvals, corrections, rejections, user instructions, repeated edits, ignored notifications, accepted suggestions, delegation patterns, and response behavior.

Explicit instructions are stronger than inferred behavior. Inferences remain provisional until confirmed.

## 9. Autonomy and trust

### Automatic

- Low-risk internal reminders
- Routine status checks
- Approved follow-up messages
- Task creation
- Daily summaries
- Low-risk scheduling
- Updates to connected task systems

### Confirmation required

- Ambiguous ownership
- Changed deadlines
- Important internal messages
- Manager escalations
- Shared project updates

### Human approval required

- External communication
- Customer or partner commitments
- Financial, legal, HR, or hiring actions
- Sensitive information
- Irreversible actions
- High-impact decisions

The system must not guess when uncertainty could create operational, reputational, or safety risk.

Required controls: explicit user and administrator permissions, role-based access, visible source context, confidence scores, a complete action audit trail, correction and undo where possible, and retention and deletion controls for audio and transcripts. Unauthorized actions are not tolerated at any rate.

## 10. Architecture

```text
Sarvam speech, language, translation, and voice capabilities
                         ↓
Context, memory, permissions, and policy layer
                         ↓
Intent, commitment, risk, and attention classification
                         ↓
Action and workflow execution layer
                         ↓
Connected company tools
                         ↓
Exception inbox and human feedback
```

See [`LLD.md`](LLD.md) for the detailed design.

## 11. Out of scope

- A replacement for Slack, email, or project-management tools.
- A meeting-notes product.
- A generic autonomous AI employee.
- Employee surveillance or continuous monitoring of private conversations.
- Autonomous external communication.
- Sensitive finance, HR, legal, hiring, or customer decisions.
- Every department and workflow at once.
- Every Indian language on day one.
- A standalone telephony or WhatsApp platform.
- A custom foundation model.
- A universal knowledge graph before the workflow is validated.
- Dozens of integrations before one workflow shows measurable value.
- Automation that increases message volume.

## 12. Pilot plan

### Phase 0 — Discover

Interview 8–12 Strategy & Operations users. Map recurring coordination work, current tools, delays, approval patterns, and the baseline time spent on follow-ups.

### Phase 1 — Observe

Process approved inputs in read-only mode. Extract actions and propose owners, deadlines, messages, and escalation rules. No autonomous actions.

### Phase 2 — Assist

Allow users to approve messages, create tasks, request updates, and schedule reminders. Add audit logs, corrections, and user feedback.

### Phase 3 — Execute safely

Allow low-risk internal actions to run automatically when permissions, confidence, and reversibility thresholds are satisfied.

### Phase 4 — Expand

Validate with Partnerships or Sales. Only then decide whether to make this an internal operating layer, a Sarvam reference application, or a broader platform capability.

## 13. Success metrics

### Outcome metrics

- Reduction in time spent on routine follow-ups.
- Reduction in overdue commitments.
- Reduction in ownerless actions.
- Faster time from conversation to execution.
- Percentage of routine communication handled automatically.
- Percentage of escalations that genuinely require human attention.
- User-reported reduction in cognitive load.

### Quality and trust metrics

- Correct owner assignment rate.
- Correct deadline and intent extraction rate.
- False escalation rate.
- User correction and override rate.
- Unauthorized action rate: target zero.
- Audio or transcript privacy incidents: target zero.

## 14. Intern-sized deliverable

The project should deliver one vertical slice rather than the whole vision:

```text
One multilingual communication source
→ structured commitments
→ confirmed follow-up
→ status tracking
→ completion verification
→ overdue escalation
→ human-attention briefing
```

The work that matters most: identify the right internal workflow, design the context and permission model, dogfood Sarvam's capabilities, measure before-and-after impact, and produce a prototype that Sarvam could show to an enterprise customer. [`BUILD_SPEC.md`](BUILD_SPEC.md) describes how to build it.
