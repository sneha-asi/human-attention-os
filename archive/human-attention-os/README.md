# Human Attention OS

A context-aware layer that turns communication into execution. It handles routine coordination (follow-ups, scheduling, status collection, reminders) in the background and brings a human in only when the work needs judgment.

**Status:** design documents only. No code has been written yet.

## The idea

As AI takes over more communication and routine execution, human attention becomes the scarce resource. Most coordination work, such as chasing updates, finding owners, and rescheduling, does not need a human at all.

The system takes voice or text in the language people already use, including code-mixed speech, and resolves it against what it knows: who is speaking, what they own, the projects and commitments involved, and what it is permitted to do. It then does one of five things:

1. handles the request itself;
2. coordinates with the right people or systems;
3. creates and follows an action through to completion;
4. asks one focused clarification question;
5. hands the item to a human, with context, because it involves judgment, ambiguity, or risk.

```text
Communication → context → action → follow-through
                             ↓
                 human attention for exceptions
```

## Example

> "The customer wants the enterprise demo moved to next week. Check availability, reschedule it, update the relevant record, and tell the account owner."

The system checks calendars, reschedules, updates the record, and notifies the owner. It interrupts a human only if there is a conflict or a decision it is not permitted to make.

## Why Sarvam

Sarvam's Indian-language speech recognition, translation, transliteration, language identification, text-to-speech, and voice agents make this usable by people who do not work in English or at a keyboard. The models are the foundation; the product is the context, permissions, action layer, and follow-through built on top of them.

## Documents

| Document | Contents |
| --- | --- |
| [`PRD.md`](PRD.md) | Problem, first users, MVP scope, autonomy rules, pilot plan, metrics |
| [`LLD.md`](LLD.md) | Services, data model, decision engine, APIs, reliability, evaluation |
| [`BUILD_SPEC.md`](BUILD_SPEC.md) | Stack, components, and build order for the first vertical slice |
