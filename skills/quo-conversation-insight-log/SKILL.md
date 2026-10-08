---
name: quo-conversation-insight-log
description: Create a bounded, passive, prioritized insight log from recent Quo messages, transcribed calls, missed calls, and voicemail transcripts through the Quo MCP. Use for call-log templates, recent customer-conversation reviews, daily or weekly communication digests, and finding important requests, risks, opportunities, commitments, or follow-ups in Quo with ChatGPT or Claude. Do not use it to claim complete call analytics, call volumes, or coverage of answered calls without transcripts.
---

# Quo Conversation Insight Log

Build a decision-useful log of recent customer conversations. Treat this as qualitative conversation review, not an analytics export.

Before running, read [prioritization.md](prioritization.md) and [output-template.md](output-template.md).

## Set the window

1. Use the user's explicit start and end when provided.
2. Otherwise, set `since` to 00:00 seven calendar days before the run in the user's timezone and set `through` to the current time.
3. If the user asks for changes since a previous run, use the previous log's exact `through` timestamp. If it is unavailable, use the seven-day default and disclose the fallback.
4. State the human-readable window, timezone, and exact ISO-8601 boundaries at the top of the result.
5. Never silently query all history. Do not default to more than seven days.

Example default for a run on July 24, 2026 in America/Toronto:

```text
Since: 2026-07-17T00:00:00-04:00
Through: <current time in America/Toronto>
```

## Read Quo safely

Use the Quo MCP tools by capability; exact tool prefixes may differ between ChatGPT and Claude.

**Run every Quo fetch sequentially. Never issue parallel, concurrent, batched, or `Promise.all`-style MCP requests. Only one inbox and one fetch operation may be active at a time.**

1. Call `list-inboxes` to discover accessible inboxes. Respect an inbox or owner filter from the user; otherwise review all accessible inboxes.
2. Choose the first inbox. Do not start work on any other inbox yet.
3. Call `fetch-messages` for that inbox with `createdAfter` and `createdBefore`. When `participantPhoneNumber` is omitted, do not rely on `maxResults`: the whole-inbox handler ignores it. Review and normalize the result before making another MCP call.
4. Interpret whole-inbox message coverage signals exactly:
   - `[next_conversations: TOKEN]`: continue sequentially with `conversationPageToken: TOKEN`.
   - `[incomplete: N more participant(s) ...]`: there is no pagination token for the omitted participants. Narrow the time window or query a known high-priority contact with `participantPhoneNumber`; otherwise disclose incomplete participant coverage.
5. Finish or stop the inbox's message stream before proceeding. Do not treat `[incomplete]` as a `conversationPageToken` signal.
6. Call `fetch-call-transcripts` for the same inbox and window with `maxResults: 100`. Review and normalize the result before making another MCP call. Follow `conversationPageToken` for older discovery batches; use `participantPhoneNumber` with `pageToken` only for a requested or high-priority contact drill-down.
7. Call `fetch-missed-calls` for the same inbox and window as a third sequential stream when the tool is available. Include useful voicemail transcripts and preserve returned recording URLs as sources. Pass only arguments supported by the active tool schema and follow its returned pagination or coverage signals exactly. If the tool is unavailable, state that missed calls and voicemails were not reviewed.
8. Finalize and compact that inbox's candidates. Discard full raw transcript and message text that is no longer needed.
9. Only then move to the next inbox and repeat steps 3–8. Do not prefetch, overlap, or queue calls for later inboxes.
10. Default to at most three tokenized pages per stream and inbox; stop earlier once the boundary is covered. If the window is still incomplete, disclose the cap and offer a narrower inbox, owner, contact, or date window.

Recover from weight or response-size limits without parallelism:

- For whole-inbox `fetch-messages`, do not lower `maxResults`; it has no effect. Split the date range into smaller, non-overlapping intervals and process those intervals sequentially, newest first. If `[incomplete]` persists, narrow again or query a known contact.
- For `fetch-call-transcripts`, retry only the failing inbox and stream with `maxResults: 25`, then paginate sequentially.
- For `fetch-missed-calls`, use only the active tool's documented volume controls. If necessary, split the date window sequentially.

Do not send messages, alter contacts, or perform any other write. This is a passive, read-only skill.

## Normalize and deduplicate

Represent each candidate with:

- channel: message, transcribed call, missed call, or voicemail
- date and time in the user's timezone
- contact or recognizable participant; omit full phone numbers by default
- inbox and team member when available
- one-sentence outcome or issue
- explicit ask, commitment, deadline, risk, or opportunity
- source conversation/call identifier when returned by Quo

Merge messages and calls that concern the same contact and topic into one item. Keep the newest state, while preserving an earlier unfulfilled commitment or deadline. Do not create separate entries for greetings, confirmations, or repeated fragments of one exchange.

Prefer a contact name already returned by Quo. For a high-priority candidate with a returned contact ID, call `get-contact` sequentially to enrich the label. If only a phone number is available, keep it masked by default. Do not enumerate the entire contacts directory merely to replace every phone number. When the user explicitly requests full contact naming, use `list-contacts` and `get-contact` sequentially and stop after the needed matches are found.

## Prioritize the log

Apply the rubric in [prioritization.md](prioritization.md). Use three approximate bands:

- **P1 — act now:** time-sensitive request, escalation, churn or safety risk, high-value blocker, missed commitment, or customer waiting on a decision.
- **P2 — follow up:** concrete opportunity, request, unresolved question, promised action, or meaningful product/service friction without immediate urgency.
- **P3 — keep in view:** useful relationship context, early signal, repeated theme, or non-urgent insight worth retaining.

Order within each band by urgency, then business/customer impact, then recency. Do not invent numeric scores or imply scientific precision.

## Produce the insight log

Use [output-template.md](output-template.md). Keep entries concise and evidence-based. For each P1 or P2 item, make the next step explicit; label it as a recommendation unless the conversation contains a real commitment.

Add a short patterns section only when at least two independent conversations support the pattern. Distinguish direct evidence from inference. Never infer sentiment, intent, or an owner from weak evidence.

## Preserve the product boundary

The Quo MCP is not a complete analytics source:

- `fetch-call-transcripts` covers completed calls with usable dialogue, not every answered or untranscribed call.
- `fetch-missed-calls` adds missed-call and voicemail evidence when available, but it still does not make the result a complete call ledger.
- Cross-participant retrieval and message results are bounded and may be paginated or capped.
- Absence from the returned data does not prove that an event did not occur.
- Counts from returned records describe records reviewed, not total workspace activity.

Do not calculate or claim total call volume, answer rates, missed-call rates, conversion rates, complete agent performance, or exhaustive contact coverage. When the user needs those metrics, recommend a Quo Analytics CSV export and keep that workflow separate from this insight log.

## Cross-platform behavior

Keep the skill portable between ChatGPT and Claude:

- depend only on the Quo MCP and Markdown output
- identify tools by capability when platform-specific names differ
- avoid platform-specific filesystem, code execution, memory, or UI assumptions
- preserve the same window, coverage caveats, priority bands, and output schema on both platforms
