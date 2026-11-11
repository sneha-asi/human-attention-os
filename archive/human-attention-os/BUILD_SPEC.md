# Human Attention OS — Build Spec

## Goal

Build one working vertical slice:

```text
Voice/text input
→ Sarvam understanding
→ commitment and owner extraction
→ human/AI routing
→ approved follow-up
→ task tracking
→ exception briefing
```

## First version

### Build

- Internal web app.
- Voice and text input.
- English, Hindi, and Hinglish examples.
- Synthetic Sarvam-style organization data.
- Optional user-owned audio or text files.
- Sarvam speech, language, translation, and text-generation APIs.
- Commitment, owner, deadline, and priority extraction.
- Human approval for medium- and high-risk actions.
- Mock email, Slack, calendar, and task connectors.
- Action status, reminders, escalation, and audit history.
- Exception inbox showing only human-required work.

### Do not build

- Sarvam internal system integrations.
- Unrestricted email or Slack monitoring.
- Autonomous external communication.
- Production multi-tenant infrastructure.
- HR, finance, legal, hiring, or customer decisions.
- A new chat or project-management product.

## Stack

- Frontend: Next.js and TypeScript.
- Backend: Python and FastAPI.
- Database: PostgreSQL.
- Local development: Docker Compose.
- Sarvam calls: one `language-runtime` module.
- Connectors: mock implementations behind a shared interface.

## Core components

1. **Input layer** — accepts text, audio, and fixture data.
2. **Language runtime** — calls Sarvam and returns normalized text, language, and structured output.
3. **Context layer** — stores people, teams, projects, permissions, and policies.
4. **Attention Firewall** — chooses auto-execute, confirm, clarify, delegate, or surface.
5. **Action engine** — tracks commitments, tasks, dependencies, and evidence.
6. **Workflow runner** — handles retries, reminders, timers, and escalation.
7. **Mock connectors** — simulate communication and task systems.
8. **Exception inbox** — shows decisions, ambiguity, risks, and blocked work.
9. **Audit log** — records source, model output, policy decision, command, and result.

## Action lifecycle

```text
Proposed
→ Clarification or confirmation
→ Approved
→ Executing
→ Waiting for evidence
→ Completed / Blocked / Escalated
```

Model output must never directly trigger an external action. Only an approved command can do that.

## Policy defaults

- Low-risk, internal, reversible actions: automatic only when enabled by the user.
- Ambiguous ownership, deadlines, or important internal messages: confirmation required.
- External, sensitive, financial, legal, HR, customer, or irreversible actions: human approval required.

## Minimum data objects

- User
- Team
- Project
- Policy
- Source event
- Commitment
- Action
- Approval
- Evidence
- Attention item
- Audit event

Every action must retain its source, confidence, policy decision, and execution history.

## Build order

1. Synthetic fixtures and database schema.
2. Text input and commitment extraction.
3. Context and Attention Firewall.
4. Approval flow and exception inbox.
5. Mock connectors and action execution.
6. Reminders, evidence, and escalation.
7. Sarvam voice input and spoken briefing.
8. Optional personal-data mode.

## Definition of done

A user can submit a realistic voice or text instruction and see:

- What the system understood.
- Which owner and deadline it identified.
- Why it requires or does not require human approval.
- A follow-up sent through a mock connector.
- A task created and tracked.
- A simulated overdue escalation.
- A concise human-attention briefing.

The demo must include one ambiguity case and one correctly blocked high-risk action.

