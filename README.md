# Quo MCP for Grok Build

Connect [Quo](https://www.quo.com) (formerly OpenPhone) to Grok Build so you can work with business texts, contacts and contact notes, tasks, call transcripts, and missed calls in plain language, and share feedback with the Quo product team.

This plugin adds Quo's official hosted MCP server. It does not install or execute a local binary. On first connection, Grok opens Quo's OAuth flow in your browser so you can choose the workspace and authorize access; no API key or environment variable is required.

## What you can do

The production Quo MCP server provides these 19 tools:

| Tool | Capability |
| --- | --- |
| `list-users` | List workspace members for inbox filtering and task assignment |
| `list-inboxes` | List the phone-number inboxes available in your Quo workspace |
| `send-message` | Send an SMS to one recipient |
| `send-group-message` | Send one shared group SMS to 2–10 recipients who can see one another |
| `send-bulk-messages` | Send separate private SMS messages to 2–40 recipients, with shared text or personalized text per recipient |
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
```

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

## Authentication, data, and permissions

- The plugin connects only to `https://mcp.quo.com/mcp`.
- Authentication uses Quo's hosted OAuth flow. The plugin does not read local credentials, secrets, `.env` files, or unrelated filesystem data.
- Once authorized, Grok can read the Quo workspace data needed by the tools above and can perform write actions such as sending messages, creating or updating contacts, contact notes, and tasks, and submitting feedback to Quo. Review recipients and content before approving consequential actions. The included skills describe their own behavior, including Voicemail Triage's internal task creation after guided setup.
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
