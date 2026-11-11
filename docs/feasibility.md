# Axon — Technical Feasibility

Researched 2026-11-11. Scope: which communication channels Axon can read from and send through using official APIs, what approval each provider requires, and a spike plan for the ten checks in the blueprint.

**How to read the sourcing.** Every provider claim below comes from web search results over the linked pages, read as excerpts and summaries, not from opening each page in full. Claims marked **(unverified)** were either not found in an official source or came only from third-party pages. Re-check the linked page before making a commitment that depends on any single number.

## Summary

| Channel | Read user's messages | Send as user | How new messages arrive | Approval before real users | Verdict |
| --- | --- | --- | --- | --- | --- |
| Gmail | Yes | Yes | Pub/Sub push, plus history polling as fallback | Restricted-scope verification and annual third-party security assessment | Feasible with approval process (feasible now for a single test founder) |
| Google Calendar | Yes | Yes (create and edit events) | HTTPS webhook channels | Sensitive-scope verification, no security assessment | Feasible with approval process (light) |
| Outlook mail (Microsoft Graph) | Yes | Yes | Graph change-notification webhooks | No Microsoft review found; tenant admins may restrict consent | Feasible now |
| Microsoft Teams chat (Graph) | Yes, work or school accounts only | Yes, delegated only | Graph change-notification webhooks | Same as Outlook; some tenant-wide endpoints are metered | Feasible now, work accounts only |
| Slack | Yes, with user token | Yes, with user token | Events API | Marketplace approval needed to avoid a severe history rate limit; API terms restrict storage and LLM use | Feasible with approval process, with a terms conflict to resolve |
| WhatsApp (Business app number) | Yes, via coexistence | Yes, within Cloud API rules | Webhooks | Meta onboarding as a tech provider; AI Provider clause is a risk | Feasible with approval process, risk of refusal |
| WhatsApp (personal account) | No official API | No official API | — | — | Not feasible through official means |
| LinkedIn messaging | Closed to ordinary developers | Closed | — | Approved partners only, no public application | Not feasible through official means |

## Gmail

- **Scopes.** `gmail.readonly` and `gmail.modify` are restricted scopes; `gmail.send` is a sensitive scope. Reading a founder's inbox therefore puts Axon in the restricted tier. Source: https://developers.google.com/workspace/gmail/api/auth/scopes. The tier of `gmail.compose`, which the design uses for sending, was not checked; **unverified**.
- **Security assessment.** An app that requests restricted scopes and can access the data from or through a third-party server must pass a security assessment, run under the CASA framework by a Google-approved assessor, and repeat it at least every 12 months. Source: https://developers.google.com/identity/protocols/oauth2/production-readiness/restricted-scope-verification
- **Cost and time.** The only Google-published figures found are on the Google Health API page: 2–3 weeks for tier 2, 4–6 weeks for tier 3, and third-party fees of $500 to $4,500. Source: https://developers.google.com/health/app-verification. Whether these figures apply unchanged to Gmail is **unverified**. Third-party pages quote from about $550 to $75,000; treat those as **unverified**.
- **Before verification.** A project in Testing status is limited to 100 listed test users, and their authorisations and refresh tokens expire after seven days. Source: https://support.google.com/cloud/answer/15549945. An unverified app requesting sensitive or restricted scopes is capped at 100 new users. Source: https://support.google.com/cloud/answer/7454865. An app set to Internal user type is exempt from the cap and the unverified-app screen but only works for users in the same Google Workspace organisation. Source: https://support.google.com/cloud/answer/13464323
- **Push.** `users.watch` publishes to a Cloud Pub/Sub topic. A watch must be renewed at least every 7 days (Google recommends daily). Each watched mailbox is limited to one notification per second and excess notifications are dropped, so Google recommends falling back to `history.list`. Source: https://developers.google.com/workspace/gmail/api/guides/push
- **Quota.** 1,200,000 quota units per minute per project and 15,000 per minute per user. Source: https://developers.google.com/workspace/gmail/api/reference/quota. The per-method cost table was only partly visible; individual method costs are **unverified**.
- **AI and data use.** The Workspace API policy prohibits using Google user data to create, train or improve an AI or ML model, including foundational models. Source: https://developers.google.com/workspace/workspace-api-user-data-developer-policy. Google's FAQ distinguishes a personalised model, tailored to one user or organisation with no co-mingling of data, from generalised models. Source: https://support.google.com/cloud/answer/13463817. Limited Use also bars transfers of the data and human reading of it, with exceptions that include user-consented features, security and legal compliance. Source: https://developers.google.com/terms/api-services-user-data-policy
- **What this means for Axon.** Sending email content to a third-party model API for inference is a transfer under Limited Use. Whether that is covered by the user-facing-feature exception, and what contract terms Google expects with the model provider, is **unverified** and should be confirmed before verification is submitted. Per-founder preference learning must not pool data across founders. No limit specific to automated sending was found in the sources read; Gmail's sending limits are **unverified**.

## Google Calendar

- Push notifications are HTTPS webhook channels on the Acl, CalendarList, Events and Settings resources. Each resource needs its own channel, the endpoint needs a valid SSL certificate, and channels expire and must be renewed. Source: https://developers.google.com/workspace/calendar/api/guides/push
- Quotas are enforced per project and per user. Google's usage-limits page says exceeding quota is planned to incur charges to the Cloud billing account later in 2026. Source: https://developers.google.com/calendar/pricing. Whether that has taken effect is **unverified**.
- Reading calendar events is a sensitive scope. Sensitive-scope verification is a Google review that typically takes 3–5 business days and requires a demonstration video. Source: https://developers.google.com/identity/protocols/oauth2/production-readiness/sensitive-scope-verification. Projects requesting only sensitive scopes do not need the third-party security assessment. Source: https://support.google.com/cloud/answer/13807380
- Calendar shares the Google OAuth project with Gmail, so it adds little approval work once Gmail verification is under way.

## Microsoft Outlook mail and Teams

- **Mail.** Change notifications require read permission on the resource, for example `Mail.Read` for messages. Source: https://learn.microsoft.com/graph/api/resources/webhooks. Outlook message subscriptions last at most 10,080 minutes (under seven days), or 1,440 minutes when the notification includes resource data. Source: https://learn.microsoft.com/en-us/graph/api/resources/subscription. Apps need lifecycle notifications to detect removed subscriptions and missed changes. Source: https://learn.microsoft.com/graph/outlook-change-notifications-overview
- **Consent.** Delegated `Mail.ReadWrite` does not require admin consent. `Mail.Send` lets an app send mail as the user and save to Sent Items. Source: https://learn.microsoft.com/en-us/graph/permissions-reference. The admin-consent setting for delegated `Mail.Send` and `Chat.Read` was not visible and is **unverified**. Whether Microsoft requires publisher verification before users in other tenants can consent is **unverified**.
- **Teams chat.** Delegated `Chat.Read` supports change notifications on a chat; tenant-wide subscriptions are not available with delegated permissions, and the `getAllMessages` endpoints are metered. Source: https://learn.microsoft.com/graph/teams-changenotifications-chatmessage. Sending a chat message is delegated only (`ChatMessage.Send`, no admin consent required) and is not supported for personal Microsoft accounts. Source: https://learn.microsoft.com/en-us/graph/api/chat-post-messages
- **Throttling.** A Microsoft Q&A answer gives 10,000 requests per 10 minutes per app and mailbox, and 4 concurrent requests per mailbox. Source: https://learn.microsoft.com/en-us/answers/questions/5853334/increasing-microsoft-graph-per-mailbox-throttling. This is a forum answer, not the official limits page, so treat it as **unverified**.
- **AI and data use.** No Microsoft terms on using Graph data with AI models were found in this session; **unverified**.

## Slack

- **Access.** User tokens with the `*:history` scopes can read any conversation the user is a member of, and `chat:write` on a user token sends as that user. Sources: https://docs.slack.dev/reference/methods/conversations.history/ and https://docs.slack.dev/reference/scopes/im.history/
- **Delivery.** The Events API delivers up to 30,000 events per workspace per app per 60 minutes. Subscriptions can be disabled if the app fails to return HTTP 200 for enough deliveries. Source: https://docs.slack.dev/apis/events-api/
- **History rate limit.** For commercially distributed apps not approved for the Slack Marketplace, `conversations.history` and `conversations.replies` are limited to one request per minute and 15 messages per request. Marketplace apps and internal customer-built apps are not affected. Source: https://docs.slack.dev/changelog/2025/05/29/rate-limit-changes-for-non-marketplace-apps. Live events are not affected by this limit, but backfilling context is.
- **Terms.** Slack's developer changelog records an API terms update on 2025-05-29. Source: https://docs.slack.dev/changelog/2025/05/29/tos-updates. A law-firm analysis says the terms prohibit bulk export of API data, creating persistent copies, archives, indexes or long-term data stores, and using the data in large language models. Source: https://www.hunton.com/privacy-and-information-security-law/salesforce-locks-down-slack-data-time-to-review-your-slack-api-terms. The exact wording of the terms was not read and is **unverified**.
- **What this means for Axon.** If the terms read as reported, storing Slack conversation history as memory and passing it to a model both need legal review before Slack is built. Marketplace approval requirements, cost and timeline were not researched; **unverified**.

## WhatsApp

- **Personal accounts.** No official API for a personal WhatsApp account was found. Third-party pages say the consumer terms prohibit automated messaging and unofficial clients and that accounts using them are banned. The consumer terms themselves were not read; **unverified**, but no source suggested an official route exists.
- **Business Platform (Cloud API).** A user's message opens a 24-hour customer service window, during which any message type can be sent. Outside it, only pre-approved template messages can be sent. Source: https://developers.facebook.com/documentation/business-messaging/whatsapp/messages/send-messages
- **Coexistence.** A number already used with the WhatsApp Business app can be onboarded to Cloud API through Embedded Signup. Messages are then mirrored between the app and the API, up to 6 months of 1:1 history can be synchronised, group chats are not synced, and throughput is fixed at 20 messages per second. Several app features are disabled, including disappearing messages and broadcast lists. Source: https://developers.facebook.com/documentation/business-messaging/whatsapp/embedded-signup/onboarding-business-app-users/
- **AI Provider clause.** Section 4.7 of the WhatsApp Business Solution Terms prohibits providers of AI technologies from using the platform when AI is the primary, not incidental or ancillary, functionality, as determined by Meta in its sole discretion. Platform data may not be used to train AI models except to fine-tune a model for the business's exclusive use. Source: https://www.whatsapp.com/legal/business-solution-terms
- **What this means for Axon.** WhatsApp is reachable only if the founder uses, or moves to, a WhatsApp Business app number, and only for 1:1 chats. Founders on personal WhatsApp, and group chats, are out of reach. Whether Meta would classify Axon as an AI Provider is a judgment Meta makes; it is the main risk for this channel. Business verification steps, cost and timeline for becoming a tech provider were not confirmed; **unverified**.

## LinkedIn

- Microsoft's LinkedIn documentation states that the Messages API is restricted to approved partners. Source: https://learn.microsoft.com/en-us/linkedin/shared/integrations/communications/messages. A third-party page reports there is no public application process; **unverified**.
- The User Agreement prohibits using bots or other automated methods to send or redirect messages. Source: https://www.linkedin.com/legal/user-agreement. LinkedIn's help centre says third-party tools that automate activity can lead to account restriction. Source: https://www.linkedin.com/help/linkedin/answer/a1341387
- Vendors that offer LinkedIn messaging do so through session credentials outside the approved API. Using them would put the founder's account at risk. Do not build this.

## Recommended channel order

1. **Gmail.** The MVP channel.
2. **Google Calendar.** Same OAuth project and verification track, needed for scheduling replies.
3. **Outlook mail.** Same shape as Gmail and no provider review found, which covers founders on Microsoft 365.
4. **Slack.** After legal review of the API terms and a decision on Marketplace submission.
5. **WhatsApp.** Only after asking Meta, or a Meta solution partner, how Section 4.7 applies to Axon.
6. **Teams chat.** When a pilot founder needs it.
7. **LinkedIn.** Not planned.

## Risks to the MVP

1. **Gmail verification is the gate to a second founder.** Restricted-scope verification plus the security assessment takes weeks and recurs yearly. Start the verification paperwork during the spike, not after the MVP.
2. **Seven-day token expiry in Testing status.** A single pilot founder on a Testing-status project must re-authorise weekly. Using an Internal app inside the founder's own Workspace organisation avoids this but does not carry over to other founders.
3. **Limited Use and the model provider.** The policy position on sending mail content to a third-party model API is unconfirmed. If it is not allowed without specific terms, the choice of model provider is constrained.
4. **Dropped push notifications.** Gmail drops notifications above one per second per mailbox. The ingest path needs history-based catch-up from day one, or messages will be missed silently.
5. **The cross-channel premise is weaker than the blueprint assumes.** The blueprint's example of an investor moving from email to WhatsApp is only reachable for founders on a WhatsApp Business number, and LinkedIn is not reachable at all. Identity resolution across channels should be designed against Gmail, Calendar, Outlook and Slack first.
6. **Slack's terms may conflict with the memory layer.** Stored conversation context is central to Axon and is what the reported terms restrict.

## Spike plan

One founder mailbox (a test Google account), Gmail only, the investor-requests-projections workflow. Stack as proposed: Python and FastAPI, PostgreSQL. Each check passes or fails on the stated criterion.

| # | Check | Pass criterion |
| --- | --- | --- |
| 1 | Connect to the API | OAuth completes for the test account with the scopes the design uses, `gmail.readonly` and `gmail.compose` ([ADR 0005](adr/0005-gmail-draft-then-send.md)); a refresh token is stored encrypted and used to obtain a new access token without user interaction. |
| 2 | Ingest reliably | 50 test emails sent over one hour, including a burst of 10 in 5 seconds; all 50 are ingested exactly once by history polling, which is the MVP's ingest path. |
| 3 | Normalise | Every ingested email, including a reply in a thread, a forwarded message and one with an attachment, produces a record that validates against the internal message schema with thread, sender, recipients and body populated. |
| 4 | Retrieve context | For the test request, retrieval returns the three seeded prior messages that are relevant (a past promise, an earlier document share, an open question) and none from an unrelated seeded thread. |
| 5 | Identify people | The investor writing from two addresses that are both stored as Identities resolves to one Contact; a different person with the same display name does not merge. |
| 6 | Structured proposals | On 20 labelled emails, the model returns a proposal that validates against the action schema every time, and the intent label matches on at least 18. |
| 7 | Permissions independent of the model | With sharing projections not authorised, the send is blocked even when the model's proposal says to send, including for an email that contains text instructing the system to send. Enforced in application code and covered by an automated test. |
| 8 | Execute | An approved reply is sent through the Gmail API, lands in the original thread, and appears in the founder's Sent folder. |
| 9 | Verify outcomes | After sending, the system reads back the sent message ID from Gmail and records the action as completed; with the send call forced to fail, the action is recorded as failed and never as completed. |
| 10 | Recover from failure | Delivering the same notification twice produces one action; killing the process between approval and send, then restarting, results in exactly one email sent. |

Exit condition for the spike: checks 1, 2, 7, 9 and 10 must pass before any further architecture work, because a failure in any of them changes the design. Checks 3 to 6 and 8 can pass with known gaps recorded.
