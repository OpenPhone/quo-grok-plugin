# SMS conversations

Use this when the user wants Sona to pursue a goal through back-and-forth texting. For one dictated text or a one-way reminder, use `send-message`; starting Sona authorizes it to compose and handle replies.

## Give Sona a bounded goal

Resolve the sending inbox with `list-inboxes` and the recipient to an E.164 number. The user must be a member of the inbox, and the workspace must have agent conversations enabled. If the user says "follow up on our call," fetch the relevant messages or call transcript sequentially and include the needed context in the task.

Include the goal, success condition, confirmed options, dates and timezone, and what to do if the person declines or none of the options works. The task is an instruction to Sona, not the first text to copy. For example:

> Ask Morgan to choose between the two delivery windows the user approved. Confirm their choice and collect access instructions. If neither works, ask for alternatives and hand back the options without promising a delivery slot. Answer business questions using the workspace's knowledge; don't invent a discount or change the order.

Use the actual approved windows in a real task; never send this example's placeholders. Keep it within 2,000 characters. Share only relevant prior context, not an entire inbox export. A special arrangement for this person belongs here, not in the shared knowledge document.

## Confirm and start

Confirm the from number, recipient, task, and limits. Explain that Sona will write the first text and keep replying autonomously. An approval to send a specific draft is not approval to switch to an autonomous conversation. Conversely, once the user approves this bounded conversation, do not ask them to approve every text Sona writes.

Call `start-agent-conversation` with `from`, `to`, `task`, and the run's `idempotencyKey` (8–128 characters). Set `userConfirmed: true` only after explicit chat approval. Keep the same key and payload on retries, including after a confirmation-required result. Never bypass a decline or an error with a different key.

Store the returned `AR…` as `agentConversationId` and identify it as an SMS run. It is not a phone-call ID or a conversation-thread ID (`CN…`). An existing run returned for the same key is not a new conversation.

## Check and stop

Use `get-agent-conversation` every few minutes while checking progress; a recipient may reply minutes or hours later. Avoid phone-call-style ten-second polling. Keep the user informed when work is still in progress, and only promise later checks or notifications when the host actually supports them.

Final statuses are `completed`, `failed`, and `canceled`. Read the failure/end reason, summary, and collected data. A completed run may have failed to achieve the requested goal. If the summary is not ready, state that rather than inventing an outcome. Use `fetch-messages` for the actual text exchange, scoped to the confirmed inbox and recipient.

Sona stops when the goal is reached, the recipient opts out, a teammate replies and takes over, the conversation is canceled or marked done, or it reaches its service limit. Do not send a competing manual reply while Sona is still working. If the user wants to take over, cancel the identified run before any separately authorized send; a teammate's own reply also takes over.

For an explicit stop request, call `cancel-agent-conversation` with the saved `agentConversationId` and report the result. Texts already sent cannot be recalled. Do not restart after STOP, takeover, cancellation, or a service limit on your own initiative.

## Resolve failures by their cause

A run may fail before sending, such as for recipient opt-out or carrier registration. Report the returned reason; do not assume every failure means missing OAuth scopes. Use `get-a2p-registration` only when its read-only status would help diagnose a reported carrier-registration blocker. Drafting or submitting a registration, including any billed submission, is separate work requiring the user's instruction and the tool's approval flow.

If starting times out, inspect the returned ID, or retry with the original key if no ID was returned. Stop repeated attempts if the state stays uncertain. A changed recipient, revised goal, or second outreach attempt is a new action to confirm, not a retry.

Source: [Quo MCP SMS conversation tools](https://www.quo.com/docs/2026-03-30/mcp/tools#sms-conversations).
