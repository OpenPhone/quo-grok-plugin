# Phone calls

Read this for starting, checking, or stopping a real phone call. Use the shared task preparation and approval guidance in `SKILL.md` before starting.

## Prepare the call

Use `list-inboxes` to resolve `from`; the user must be a member of that number. Set `to` to the confirmed recipient in international E.164 format. A name alone or an unresolved local number is not enough to dial.

Build the task around the outcome, permitted commitments, and fallback. For an appointment, include the service, acceptable actual dates and time windows, timezone, and whether Sona may book or must only collect alternatives. Do not assume a reservation is booked just because Sona can discuss availability; the result must establish the booking.

Choose voicemail handling before confirmation:

- Omit `voicemail`, or set `voicemail: "hang-up"`, to hang up if voicemail answers. Use this default unless leaving a message is requested and approved.
- Set `voicemail: "leave-message"` when authorized to leave one. `voicemailMessage` supplies the approved instruction for what to say, up to 1,000 characters. Without it, the service leaves a short callback request.
- Supplying `voicemailMessage` also implies leave-message behavior. Never accidentally change the default by adding that field, and never combine it with `voicemail: "hang-up"`.

`maxDurationSeconds` limits time after the call is answered: default 300 seconds, maximum 600, minimum 1. Change it when the user requests a shorter or longer call within those limits. It is not a schedule or a delay before dialing.

Each call costs 50 workspace automation credits; a call ending within 15 seconds is free. Include that in the approval request. Insufficient credits prevent dialing. Do not promise the exception or a refund before the result is known.

A useful confirmation, using resolved details, is:

> I'll have Sona call Jordan at the number we confirmed from your Sales Line, confirm order 4821's delivery window within the options you approved, and collect alternatives if neither works. It will hang up if voicemail answers. The call uses 50 automation credits; calls ending within 15 seconds are free. Shall I place it?

In a real confirmation, show the actual from/to numbers rather than the example's shorthand. Do not add a second confirmation if the user already explicitly approved the exact action and its cost.

## Start once, then inspect

Call `start-agent-call` with `from`, `to`, `task`, and the run's `idempotencyKey` (8–128 characters). Pass voicemail/duration options only as intended. Use `userConfirmed: true` only after explicit chat approval. If Quo reports that approval is still required, get that approval and retry the same call with the same key; a decline ends the attempt.

Keep the returned `AR…` as `agentCallId`. If the response says this key belongs to an earlier run, that was a replay, not a new call. Inspect that run instead of reporting that a fresh call was placed.

Use `get-agent-call` about every 10 seconds. Calls often take 30–90 seconds, but this is an expectation, not a guarantee. The terminal statuses are:

| Status | What to report |
| --- | --- |
| `completed` | The call ended; judge the actual outcome from the summary and collected data. It can still have reached a machine or failed to achieve the task. |
| `failed` | Report the returned failure reason. Do not infer that the goal was achieved or automatically redial. |
| `no-answer` or `busy` | No successful conversation established by this result. Ask before another attempt or channel change. |
| `voicemail` | Report that voicemail was reached; do not claim a message was left. |
| `voicemail-left` | Report that a message was left, without claiming the person heard it or agreed to anything. |
| `canceled` | Report cancellation; it cannot undo anything said before the call stopped. |

If the call remains non-final, keep its status visible. Do not use elapsed time alone as evidence that it failed, succeeded, or should be redialed.

For a stop request, resolve the current run and call `cancel-agent-call` with `agentCallId`. This hangs up a live call; an already-ended call is returned unchanged. Report whether it stopped or had already ended.

## Recover without duplicate calls

- **Timeout or uncertain start:** if an ID was returned, inspect it. Otherwise retry the same intended call with its original key and payload. Do not invent a new key or claim that a timeout proves no call was placed. If uncertainty persists, explain it and stop creating attempts.
- **Idempotency conflict:** inspect the earlier run identified by the result. Do not force a changed task through with a new key unless the user wants and approves a separate call.
- **Missing scope:** reconnect OAuth as directed by the tool, then continue the same intended call with its original key. Inbox membership errors require selecting a number the user can use, not reconnecting in a loop.
- **Workspace disabled, insufficient credits, or call limits:** explain the specific blocker and the next action available to the user. Do not make rapid retries, change enablement, or try another destination to bypass it.
- **Unknown run ID:** verify it from the earlier start response and workspace. Do not create another call merely because a get request failed.

Summarize the actual outcome in plain language. Keep the run ID available for follow-up or requested feedback; distinguish it from `AC…` call/message activity IDs used for task linking. Do not put an `AR…` run ID into `create-task.activityId`.

Source: [Quo MCP phone-call tools](https://www.quo.com/docs/2026-03-30/mcp/tools#phone-calls).
