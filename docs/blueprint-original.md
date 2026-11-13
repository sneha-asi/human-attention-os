# AXON — Product Vision, Architecture & Engineering Blueprint

> This is the original blueprint, kept word for word as the source. The current plan is [`blueprint.md`](blueprint.md), whose section 10 lists what changed and why.

## 1. The Core Idea

**Axon is an AI-powered autonomous communication system, starting with founders.**

The long-term vision is to build an operating layer that automates professional communication and execution, bringing humans into the loop only when their attention, judgment, or decision is required.

### The fundamental insight

Historically:
- Work comes in.
- Humans understand the work.
- Humans execute the work.

Today:
- Work comes in.
- Humans delegate the work to AI.
- AI helps execute the work.

The future:
- Work comes in.
- AI understands what needs to be done.
- AI executes the task.
- Humans are involved only when their input is necessary.

**AI capabilities are becoming abundant, but human attention is limited.**

Axon is built around solving this attention constraint.

### The long-term vision

There are fundamentally two things people spend their professional time doing:

1. Communication.
2. Execution.

If AI can perform most of these activities reliably, there is an opportunity to automate the entire workflow rather than simply assisting humans with individual tasks.

### Initial product direction

We will not attempt to build the entire operating layer immediately.

We will start by automating professional communication for founders.

Founders are an interesting starting point because:
- They handle a high volume of professional communication.
- Their communication involves high-stakes decisions.
- They interact across multiple channels.
- They frequently need to follow up, delegate, respond and coordinate.
- Their attention is particularly valuable.

Axon will bring their professional communication channels together, understand conversations in context, manage communication workflows, and execute actions within defined permissions.

Execution automation and the broader operating layer will be future expansions.

---

## 2. Product Philosophy

Axon should not be another AI wrapper or chatbot.

The objective is to build a reliable system that can independently manage communication workflows.

We should leverage existing AI models, APIs and tools rather than unnecessarily rebuilding everything from scratch.

However, the system's core capabilities must be engineered carefully.

The important differentiators are:

- Reliability.
- Security and privacy.
- Deep contextual understanding.
- Founder-specific intelligence.
- Permission-aware autonomous execution.
- Ability to manage communication end-to-end.

The moat should not depend on access to a particular AI model.

It should become stronger through accumulated contextual understanding, founder-specific preferences, trusted permissions, workflow reliability and execution history.

---

# 3. System Architecture — The 10 Layers

We discussed ten architectural layers.

These represent the major capabilities required to make Axon function as a complete system.

## Layer 1: Integration Layer

**Purpose: Connect all professional communication channels.**

This layer brings communication from different platforms into Axon.

Potential integrations:
- Gmail.
- WhatsApp.
- Slack.
- Microsoft Teams.
- LinkedIn.
- Calendar platforms.
- Other professional communication tools.

Its responsibility is to reliably ingest messages, conversations, events and updates.

The objective is to create a unified communication entry point.

## Layer 2: Data Normalisation Layer

**Purpose: Convert information from different platforms into a common format.**

Every platform structures information differently.

For example:
- Gmail has email threads.
- WhatsApp has individual messages and conversations.
- Slack has channels, threads and replies.

The normalisation layer converts these different structures into a consistent internal representation.

This allows downstream layers to work without constantly handling platform-specific formats.

## Layer 3: Data Storage Layer

**Purpose: Store only the information necessary for the system to function.**

We specifically discussed privacy-conscious storage.

The objective is not to store every conversation indefinitely.

The system should identify and retain relevant information, such as:
- Important conversations.
- Commitments and promises.
- Pending tasks.
- Relationships.
- Relevant historical context.
- Information needed for future decisions.

Attachments and other large files may require separate storage.

### Privacy considerations

Potential approaches:
- Store data in Axon's secure cloud infrastructure.
- Consider customer-controlled or customer-hosted storage options for higher-security requirements.
- Encrypt data at rest and in transit.
- Isolate each founder's data.
- Apply strict access controls.
- Establish retention and deletion policies.
- Ensure data is only accessed when necessary for an authorised task.

The important distinction: storing data in a founder's cloud does not automatically guarantee privacy. Isolation, encryption, access control and processing permissions matter just as much.

## Layer 4: Identity & Relationships Layer

**Purpose: Understand who is who across different communication platforms.**

Example:

An investor sends an email to the founder.

The same investor later messages the founder on WhatsApp.

Axon should recognise that both identities belong to the same person.

It should also understand the person's relationship with the founder.

Possible relationship categories:
- Investors.
- Customers.
- Employees.
- Co-founders.
- Partners.
- Vendors.
- Other professional contacts.

This prevents Axon from treating the same person as multiple unrelated contacts.

## Layer 5: Context & Memory Layer

**Purpose: Understand the complete history behind a conversation.**

Axon shouldn't respond to messages in isolation.

It should retrieve relevant historical context.

For example:
- What did the founder promise this investor?
- Was a document already shared?
- Is there an unresolved question?
- Was a follow-up promised?
- What happened in the previous conversation?

This layer ensures that Axon understands the story behind a message.

## Layer 6: Intelligence Layer

**Purpose: Understand what incoming communication actually means.**

This layer interprets messages and identifies:
- Intent.
- Urgency.
- Requests.
- Commitments.
- Decisions.
- Questions.
- Required actions.
- Potential risks.

For example, if an investor asks for financial projections, the intelligence layer identifies what is being requested.

It understands the message but does not independently decide whether it is authorised to send the projections.

## Layer 7: Decision & Orchestration Layer

**Purpose: Decide what should happen next and coordinate the workflow.**

This layer determines:
- Should Axon respond?
- Should it follow up?
- Does the founder need to approve?
- Should the system wait?
- Is additional information required?
- Should the task be escalated?

It also coordinates multi-step workflows.

For example, if an investor requests financial projections, this layer checks whether the necessary information exists and whether sharing it has been authorised.

It creates the execution plan.

## Layer 8: Execution Layer

**Purpose: Actually perform the approved action.**

This is where the system executes decisions.

Examples:
- Sending emails.
- Responding to messages.
- Scheduling meetings.
- Sending documents.
- Initiating follow-ups.
- Updating communication records.

Execution must respect permissions and approval requirements.

## Layer 9: Outcome Tracking Layer

**Purpose: Verify what happened after execution.**

The system shouldn't assume an action succeeded simply because it attempted it.

This layer checks:
- Was the message successfully sent?
- Was the meeting scheduled?
- Did the recipient respond?
- Is the conversation still pending?
- Is another follow-up required?

This closes the communication loop.

## Layer 10: Founder Intelligence Layer

**Purpose: Understand how an individual founder thinks, communicates and makes decisions.**

This is potentially one of Axon's strongest differentiators.

Two founders may receive exactly the same message but respond very differently.

One founder might prefer a short, direct response.

Another might prefer a detailed explanation.

One might automatically approve certain follow-ups, while another might want to review everything.

Founder Intelligence learns:
- Communication preferences.
- Tone and writing style.
- Decision-making patterns.
- Priorities.
- Risk tolerance.
- Delegation preferences.
- Approval boundaries.
- Behavioural patterns derived from permitted interactions and feedback.

Over time, Axon should become increasingly capable of acting in accordance with the founder's preferences and judgment.

This is intended to be a major source of product differentiation.

---

# 4. Cross-Cutting Capabilities

These are not additional sequential layers.

They operate across all ten architectural layers.

## A. Security & Governance

Security must be built into the entire system.

Responsibilities:
- Authentication.
- Authorisation.
- Data encryption.
- Tenant isolation.
- Permission management.
- Approval workflows.
- Audit logs.
- Data retention and deletion.
- Secure handling of sensitive information.

The objective is to ensure Axon never accesses or executes something it isn't authorised to.

## B. Reliability & Evaluation

This is particularly important because Axon will eventually perform high-stakes communication on behalf of founders.

The system must be continuously tested.

Responsibilities:
- Test datasets.
- Scenario-based evaluations.
- Expected-output comparisons.
- Intent classification evaluations.
- Response quality evaluations.
- Follow-up evaluations.
- Regression testing.
- Failure detection.
- Behavioural improvements.

### How to build the reliability layer

We discussed the distinction between demo data and test cases.

**Demo data:** Realistic conversations and communication environments used to simulate founder workflows.

**Test cases:** Specific scenarios with defined expected behaviour and outcomes.

For example:

A founder has an investor conversation across email and WhatsApp.

The investor requests a financial document.

A test case should establish:
- What messages exist.
- What context Axon should retrieve.
- What intent it should identify.
- What response it should generate.
- Whether approval is required.
- What action it should take.
- What constitutes a correct outcome.

A collection of these scenarios becomes an evaluation dataset.

When Axon behaves incorrectly, the failure should be converted into a new test case so that the same problem does not repeatedly occur.

The evaluation infrastructure should be developed alongside the integration layer, not added at the end.

The initial pilot should use a combination of synthetic scenarios and consented, appropriately protected real founder data.

## C. Observability & Monitoring

The system needs visibility into what happens internally.

Responsibilities:
- Monitoring each layer.
- Tracking failures.
- Logging actions.
- Tracking execution status.
- Measuring latency.
- Monitoring API failures.
- Tracking model behaviour.
- Understanding why a particular decision was made.

Every important action should be traceable.

---

# 5. Product Engineering & Documentation

We discussed that building Axon requires more than writing code with AI coding agents.

The objective is to build a scalable system whose architecture does not deteriorate as the product grows.

The documentation should provide a clear blueprint for the engineering work.

## Core documentation

### 1. One-Pager

Explains:
- What Axon is.
- The problem.
- The fundamental insight.
- The target users.
- The product vision.
- How the product works.
- Why it matters.
- Long-term direction.

### 2. BRD — Business Requirements Document

Defines:
- Business objectives.
- Target users.
- Business requirements.
- Expected outcomes.
- Scope.
- Constraints.
- Business success metrics.

### 3. PRD — Product Requirements Document

Defines:
- Product functionality.
- User workflows.
- Features.
- User journeys.
- Functional requirements.
- Non-functional requirements.
- MVP scope.
- Acceptance criteria.

### 4. System Architecture / HLD

Defines:
- The ten architectural layers.
- Their responsibilities.
- How the layers interact.
- Technology choices.
- System boundaries.
- Data flow.
- Integration architecture.
- Infrastructure approach.

### 5. LLD — Low-Level Design

Defines how individual components will actually be implemented.

It should include:
- Component-level architecture.
- Database schemas.
- API specifications.
- Data models.
- Service interfaces.
- Internal logic.
- Error handling.
- State transitions.
- Integration details.

LLDs should be created for the relevant components before implementing them. They do not all need to be completed upfront.

---

# 6. Additional Engineering Documentation

Beyond BRD, PRD, HLD and LLD, we discussed several supporting engineering documents.

### Domain Model

Defines the important entities in Axon and how they relate.

For example:
- Founder.
- User.
- Contact.
- Identity.
- Communication channel.
- Conversation.
- Message.
- Commitment.
- Task.
- Action.
- Permission.
- Workflow.

### System Invariants

Defines rules that must always remain true.

Examples:
- One founder must never access another founder's data.
- An action cannot execute without the required permission.
- A completed action must have a verifiable outcome.
- A failed operation must not silently become a successful one.

### API & Event Contracts

Defines how components communicate.

This includes:
- Input and output structures.
- Event formats.
- API versions.
- Error formats.
- Compatibility requirements.

### Workflow State Machines

Defines the valid states and transitions of workflows.

For example:

Received → Understood → Decision Pending → Approved → Executing → Completed.

There must also be explicit paths for failures, retries, cancellations and escalations.

### Data Strategy

Documents:
- What gets stored.
- What gets retrieved.
- What gets retained.
- What gets deleted.
- Data isolation.
- Data ownership.
- Retention policies.

---

# 7. Coding-Agent Documentation: MD Files & Skills

We specifically discussed using AI coding agents such as Claude Code and Codex.

The objective is to make coding agents work within the system architecture rather than independently generating disconnected code.

## Markdown documentation

Potential files:

**AGENTS.md**
- Overall repository instructions.
- Architectural rules.
- Coding conventions.
- Testing requirements.
- Rules for modifying existing components.

**design.md**
- System design.
- Architecture decisions.
- Component responsibilities.
- Data flow.
- Design constraints.

**Architecture Decision Records (ADRs)**
- Important architectural decisions.
- Why a particular approach was selected.
- Alternatives considered.
- Consequences of the decision.

**Component-specific documentation**
- Integration layer documentation.
- Storage design.
- Identity resolution.
- Context retrieval.
- Intelligence and orchestration.
- Execution and evaluation.

These documents should evolve as the architecture evolves.

## Coding-agent skills

We discussed building reusable skills that guide agents through specialised engineering tasks.

Potential skills:

1. System architecture and design.
2. Architecture impact analysis.
3. LLD generation.
4. Database schema design.
5. API and integration development.
6. Data modelling.
7. Workflow and state-machine engineering.
8. Test-case generation.
9. Evaluation pipeline development.
10. Security review.
11. Reliability analysis.
12. Code review.
13. Documentation maintenance.

The idea is to give agents repeatable engineering procedures instead of relying entirely on prompts.

Not every skill needs to be built before coding. Create the foundational instructions first and add specialised skills when the relevant engineering work begins.

---

# 8. Important Engineering Principles

We also discussed several principles that help prevent a system from becoming fragile.

### 1. Separate AI intelligence from deterministic system control

The AI can interpret messages, propose decisions and suggest actions.

However, permissions, approvals, execution constraints and security policies should be enforced by deterministic application logic.

The LLM should not be the ultimate authority on what it is allowed to do.

### 2. Define clear architectural boundaries

Every component should have a clear responsibility.

Avoid tightly coupling every layer to every other layer.

Changes to one component should not unexpectedly break unrelated components.

### 3. Version contracts

When different components communicate, their interfaces should be explicitly defined and versioned.

This makes changes safer as the system grows.

### 4. Use automated architectural checks

Important architectural rules should be continuously checked through automated tests.

For example, a new feature should not accidentally bypass the permission system.

### 5. Build evaluation into development

Every significant failure should become a regression test.

The system should improve through measurable evaluations rather than subjective judgments alone.

### 6. Design for failure and recovery

External APIs fail. Network connections break. Models produce incorrect outputs.

The system must account for:
- Retries.
- Timeouts.
- Duplicate events.
- Partial failures.
- Idempotency.
- Recovery.
- Rollbacks.
- Human escalation.

---

# 9. What Needs to Happen Before Coding?

We agreed that the following is the working sequence.

| Step | Deliverable | Priority |
|---|---|---|
| 1 | One-pager | Now |
| 2 | BRD | Now |
| 3 | PRD | Now |
| 4 | Ten-layer architecture | Now |
| 5 | Cross-cutting architecture | Now |
| 6 | Domain model | Now |
| 7 | System invariants and boundaries | Now |
| 8 | First workflow state machine | Now |
| 9 | Initial HLD and LLD for the first workflow | Now |
| 10 | Technical feasibility research | Now |
| 11 | MVP scope and acceptance criteria | Now |
| 12 | Coding-agent instructions / AGENTS.md | Now |
| 13 | Foundational coding-agent skills | Now |
| 14 | Development environment and CI | Before implementation |
| 15 | Additional component-level LLDs | As development progresses |
| 16 | Advanced skills and design documents | As development progresses |
| 17 | Expanded evaluation datasets | Continuously |
| 18 | Infrastructure and scaling improvements | As required |

## 10. Technical Feasibility & Proof of Concept

Before committing to the complete architecture, we also identified the need for a small technical spike.

It should establish whether we can:

- Connect to the required communication APIs.
- Reliably ingest conversations.
- Normalise incoming information.
- Retrieve relevant context.
- Identify people across channels.
- Generate structured action proposals.
- Enforce permissions independently of the LLM.
- Execute authorised actions.
- Verify execution outcomes.
- Recover safely from failures.

This is not the complete product. It is a way to expose major technical constraints before investing heavily in implementation.

---

# 11. The First MVP Workflow

We need to avoid attempting all ten layers simultaneously.

The first MVP should prove one complete communication workflow.

Example:

**Scenario: An investor emails a founder requesting financial projections.**

1. Integration layer receives the email.
2. Normalisation layer converts it into Axon's common format.
3. Storage layer securely stores the necessary information.
4. Identity layer recognises the investor.
5. Context layer retrieves previous interactions.
6. Intelligence layer understands the request.
7. Decision layer determines whether approval is needed.
8. Execution layer sends the authorised response.
9. Outcome layer verifies the action and tracks the conversation.
10. Founder Intelligence applies the founder's preferences and approval boundaries throughout the workflow.

This gives us a measurable first implementation of the architecture.

---

# 12. What Comes Later?

We explicitly agreed not to over-engineer the product before real-world usage.

The following should evolve as Axon develops:

- More communication integrations.
- More complex workflows.
- Advanced founder intelligence.
- Larger evaluation datasets.
- Additional coding-agent skills.
- Detailed component LLDs.
- More advanced infrastructure.
- Customer-hosted deployment options.
- Broader execution automation.
- The eventual autonomous operating layer.

The initial goal is founder communication automation, not the entire future vision.

---

# 13. Final Product Philosophy

The most important thing we established during this conversation:

**We are not trying to build a collection of AI features. We are trying to engineer a system.**

A system that:
- Understands communication.
- Maintains context.
- Understands the founder.
- Makes appropriate decisions.
- Executes authorised actions.
- Tracks outcomes.
- Protects sensitive information.
- Improves through continuous evaluation.

The architecture, documentation, coding-agent instructions, skills and evaluation infrastructure should all work toward that objective.

The goal is to build something that can grow in complexity without becoming unreliable, insecure or impossible to maintain.

**Axon's starting point: Autonomous professional communication for founders.**

**Axon's long-term ambition: An autonomous operating layer that handles communication and execution, bringing humans in only when their attention is genuinely required.**
