# Quo MCP for Grok Build

Connect [Quo](https://www.quo.com) (formerly OpenPhone) to Grok Build so you can work with business texts, contacts and contact notes, tasks, call transcripts, and missed calls in plain language. Have Sona handle SMS conversations, manage inbox settings, keep Sona's business knowledge current, and share feedback with the Quo product team.

This plugin adds Quo's official hosted MCP server. It does not install or execute a local binary. On first connection, Grok opens Quo's OAuth flow in your browser so you can choose the workspace and authorize access; no API key or environment variable is required.

## What you can do

The production Quo MCP server provides these 27 tools:

| Tool | Capability |
| --- | --- |
| `list-users` | List workspace members for inbox filtering and task assignment |
| `list-inboxes` | List the phone-number inboxes available in your Quo workspace |
| `get-inbox-settings` | Read an inbox's name, business hours, timezone, automatic call recording, and recording-disclosure settings |
| `update-inbox-settings` | Change an inbox's name, business hours, or call recording settings after confirmation; requires inbox-owner or workspace-admin access |
| `send-message` | Send one SMS to one recipient; does not handle replies |
| `send-group-message` | Send one shared group SMS to 2–10 recipients who can see one another |
| `send-bulk-messages` | Send separate private SMS messages to 2–40 recipients, with shared text or personalized text per recipient |
| `start-agent-conversation` | Have Sona text one person and handle the back-and-forth toward a confirmed goal |
| `get-agent-conversation` | Check a Sona SMS conversation's status, outcome summary, and collected data |
| `cancel-agent-conversation` | Stop a Sona SMS conversation; texts already sent remain sent |
| `ask-knowledge-agent` | Start, continue, or check a conversation with Sona about the workspace's business knowledge; requires workspace-admin or owner access |
| `decide-knowledge-save` | Approve or decline Sona's proposed knowledge draft when direct in-client approval is unavailable |
| `create-contact` | Create a Quo contact, including workspace custom fields |
| `update-contact` | Update or clear fields on an existing contact, including workspace custom fields |
| `list-contacts` | List and filter contacts with pagination |
| `get-contact` | Retrieve one contact, including custom fields |
| `list-contact-notes` | Read a contact's notes and available attachment links, with pagination |
| `create-contact-note` | Add an internal note to a contact, with optional teammate mentions |
| `update-contact-note` | Replace an existing contact note's text, with optional teammate mentions |
| `list-tasks` | List Quo tasks with pagination |
| `create-task` | Create a task linked to an inbox, conversation, or activity |
| `update-task` | Update a task's title/description, assignment, due date, completion, or conversation link, one change per call |
| `fetch-messages` | Retrieve individual or group message history and attachment links, with date filters and pagination; optionally exclude done conversations in whole-inbox queries |
| `fetch-call-transcripts` | Retrieve call transcripts where enabled, with date and team-member filters and pagination; optionally exclude done conversations in whole-inbox queries |
| `fetch-missed-calls` | Retrieve unanswered incoming calls and available voicemail transcripts and recording links, with date and participant filters and pagination |
| `mark-conversations` | Mark up to 50 conversations as read, done, or open without sending messages |
| `submit-feedback` | Share the user's feedback about Quo or their experience with the Quo product team |

Contact notes are internal: adding or updating a note does not send anything to the contact. Notes accept 1–2000 characters; use `list-users` to find teammate IDs for mentions. `update-contact-note` replaces the full text, so read the existing note first when adding to it.

`submit-feedback` sends feedback to Quo's product team. It preserves the user's feedback and relevant context; it is not a way to message a customer.

Example requests:

```text
List my Quo inboxes.
Show the messages with +14165550123 from yesterday.
Summarize the calls I missed this morning.
Create a contact for Alex Rivera at +14165550123.
Show Alex Rivera's contact notes.
Add a note to Alex Rivera: "Prefers afternoon callbacks."
Create a follow-up task for the last missed call and assign it to Jordan.
Send Alex: "I'm running five minutes late."
Submit feedback to Quo: "I'd like to search contact notes by keyword."
Have Sona text Alex to confirm whether Tuesday at 2pm America/Toronto works for the consultation.
Show my Main Line's business hours and call recording settings.
Change Main Line's Friday hours to 9am–3pm in America/Toronto, keeping the other days unchanged.
Ask Sona to update our business knowledge: we are now closed on Sundays.
Mark the conversations we just reviewed as done.
```

## Sona conversations, knowledge, and inbox settings

Use `send-message` for one specific text. Use `start-agent-conversation` when Sona should write the texts and handle replies until a goal is reached. Confirm the sending inbox, recipient, goal, and relevant details before starting. Reuse the same `idempotencyKey` for retries of that conversation. A teammate's reply takes over from Sona, a recipient's STOP ends the exchange, and `cancel-agent-conversation` stops it explicitly. Check progress with `get-agent-conversation` every few minutes and read the texts with `fetch-messages`. A completed status alone does not establish that the goal was achieved; read the summary.

Sona's business knowledge covers the workspace's services, prices, hours, service area, policies, and FAQs. `ask-knowledge-agent` lets workspace admins and owners check or change it through a conversation with Sona. Start once per topic and keep the returned `interviewId` on follow-ups. Relay Sona's questions and the user's answers verbatim. Show the proposed draft and its new or changed prices and policies before asking for approval. If the client cannot collect approval directly, use `decide-knowledge-save` only after the user explicitly approves or declines. Nothing is saved without approval.

Read `get-inbox-settings` before proposing an inbox change. Explain the exact changes and confirm them before calling `update-inbox-settings`, because they affect live calls immediately. Omitted fields keep their current values. Business hours use the inbox's IANA timezone; use explicit times such as `0900` and `1730`, or `9am` and `5:30pm`. Automatic recording depends on the workspace's plan. Change recording disclosures only when the user asks.

Updating Sona's knowledge about business hours and changing an inbox's enforced hours are separate actions. Use the knowledge tools for what Sona tells people, and the inbox-settings tools for the phone line's configuration.

Sona's tools are in beta. Business-detail corrections belong in `ask-knowledge-agent`; feedback about how Sona performed can be shared through `submit-feedback`, including the conversation or interview ID when relevant.

## Included skills

The plugin includes three Quo workflows in `skills/`:

| Skill | What it does |
| --- | --- |
| [Quo Conversation Insight Log](skills/quo-conversation-insight-log/SKILL.md) | Reviews recent messages, transcribed calls, missed calls, and voicemails to produce a prioritized, read-only insight log. |
| [Quo 3-Month Drip](skills/quo-3-month-drip/SKILL.md) | Finds previously engaged contacts who have gone quiet and drafts personalized re-engagement texts for review. |
| [Quo Voicemail Triage](skills/quo-voicemail-triage/SKILL.md) | Reviews voicemails, creates internal follow-up tasks after guided setup, and drafts replies for review. |

Example requests:

```text
Create a prioritized insight log from my Quo conversations this week.
Find customers we haven't spoken with in three months and draft check-in texts.
Set up voicemail triage for my main Quo line.
```

Drip and Voicemail Triage require explicit approval before sending texts. Voicemail Triage stores its settings in a Quo task; recurring runs require scheduling support in the host and a configured schedule. Installing the skill does not start a schedule.

The root Agent Plugin format discovers skills from `skills/<name>/SKILL.md`. Each skill includes its supporting Markdown references and templates.

## Installation

Open `/plugins` in Grok Build, search for **Quo**, and install it from the xAI marketplace. Enable and trust the plugin so Grok can connect to the bundled MCP server.

To install a specific revision directly from GitHub:

```bash
grok plugin install OpenPhone/quo-grok-plugin@<full-commit-sha> --trust
```

Start a new Grok session after installation. The first Quo tool call opens a browser window for OAuth authorization. Use `/mcps` to inspect the connection.

Tools are discovered from the hosted MCP server; this plugin does not maintain a tool allowlist. If a client still shows an older tool list, refresh its MCP connection or start a new session. Reconnect OAuth if the server reports that the existing authorization lacks permissions for a new capability.

## Authentication, data, and permissions

- The plugin connects only to `https://mcp.quo.com/mcp`.
- Authentication uses Quo's hosted OAuth flow. The plugin does not read local credentials, secrets, `.env` files, or unrelated filesystem data.
- Once authorized, Grok can read the Quo workspace data needed by the tools above and can perform write actions such as sending messages, creating or updating contacts, contact notes, and tasks, and submitting feedback to Quo. Review recipients and content before approving consequential actions. The included skills describe their own behavior, including Voicemail Triage's internal task creation after guided setup.
- Sona can send and reply to texts on the business's behalf after the user confirms the conversation. Inbox-settings changes and publishing Sona knowledge require confirmation and the applicable owner or admin permissions.
- A group message exposes every recipient's phone number and replies to the whole group and cannot be undone. Use `send-bulk-messages` when recipients should receive separate private messages.
- Data returned by Quo is processed in your Grok session and is subject to the terms and privacy policies of the services you use.
- Quo may collect service usage and diagnostic events as described in the [Quo Privacy Policy](https://www.quo.com/privacy).

Sending messages requires prepaid Quo credits and counts as API messaging. Call transcripts require a plan and phone-number configuration that supports transcription.

## Support and resources

- [Quo MCP documentation](https://support.quo.com/core-concepts/integrations/mcp)
- [Quo Privacy Policy](https://www.quo.com/privacy)
- [Quo Support](https://support.quo.com)
- [Report a plugin issue](https://github.com/OpenPhone/quo-grok-plugin/issues)

## License

Apache License 2.0. See [LICENSE](LICENSE).
