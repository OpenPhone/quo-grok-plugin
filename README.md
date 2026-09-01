# Quo MCP for Grok Build

Connect [Quo](https://www.quo.com) (formerly OpenPhone) to Grok Build so you can work with business texts, contacts, tasks, call transcripts, and missed calls in plain language.

> [!WARNING]
> Quo MCP is **not HIPAA compliant**. Do not use it to transmit protected health information (PHI).

This plugin adds Quo's official hosted MCP server. It does not install or execute a local binary. On first connection, Grok opens Quo's OAuth flow in your browser so you can choose the workspace and authorize access; no API key or environment variable is required.

## What you can do

| Tool | Capability |
| --- | --- |
| `list-users` | List workspace members for inbox filtering and task assignment |
| `list-inboxes` | List the phone-number inboxes available in your Quo workspace |
| `send-message` | Send an SMS to one recipient |
| `send-group-message` | Send one shared group SMS to 2–10 recipients who can see one another |
| `send-bulk-messages` | Send separate private SMS messages to 2–40 recipients |
| `create-contact` | Create a Quo contact |
| `update-contact` | Update or remove fields on an existing contact |
| `list-contacts` | List and filter contacts with pagination |
| `get-contact` | Retrieve one contact, including custom fields |
| `list-tasks` | List Quo tasks with pagination |
| `create-task` | Create a task linked to an inbox, conversation, or activity |
| `update-task` | Update assignment, due date, completion, content, or conversation links |
| `fetch-messages` | Retrieve message history with date and participant filters |
| `fetch-call-transcripts` | Retrieve call transcripts where enabled |
| `fetch-missed-calls` | Retrieve missed incoming calls and available voicemail details |

Example requests:

```text
List my Quo inboxes.
Show the messages with +14165550123 from yesterday.
Summarize the calls I missed this morning.
Create a contact for Alex Rivera at +14165550123.
Create a follow-up task for the last missed call and assign it to Jordan.
Send Alex: "I'm running five minutes late."
```

## Installation

Once the plugin is listed in the xAI marketplace, open `/plugins` in Grok Build, search for **Quo**, and install it. Enable and trust the plugin so Grok can connect to the bundled MCP server.

For testing before marketplace approval, install a pinned revision directly from GitHub:

```bash
grok plugin install OpenPhone/quo-grok-plugin@<full-commit-sha> --trust
```

Start a new Grok session after installation. The first Quo tool call opens a browser window for OAuth authorization. Use `/mcps` to inspect the connection.

## Authentication, data, and permissions

- The plugin connects only to `https://mcp.quo.com/mcp`.
- Authentication uses Quo's hosted OAuth flow. The plugin does not read local credentials, secrets, `.env` files, or unrelated filesystem data.
- Once authorized, Grok can read the Quo workspace data needed by the tools above and can perform explicit write actions such as sending messages and creating or updating contacts and tasks. Review recipients and content before approving consequential actions.
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
