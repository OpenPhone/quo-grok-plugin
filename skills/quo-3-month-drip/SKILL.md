---
name: quo-3-month-drip
description: Find contacts a business actually spoke with but has not heard from in a while, then draft a personalized, season-aware re-engagement text for each one using the Quo MCP. Use when someone asks who has gone quiet, who they have not talked to in months, who to follow up with, who to win back, or for a re-engagement, reactivation, or check-in campaign from their Quo history. Triggers include "who haven't we talked to lately", "who went cold", "flag contacts we've lost touch with", "draft follow-ups to old customers", "reactivation list", and "who should I check in with". Do not use it to claim total call volume, answer rates, or complete contact coverage, and do not use it to send messages automatically.
---

# Quo 3-Month Drip

Produce a short, ranked list of contacts who had a real conversation with the business and then went quiet, each with a drafted follow-up text that references what was actually discussed and what is happening in their season right now.

Three rules shape everything below:

- **Dormancy is measured on human activity only.** Automated outbound never resets the clock. See Step 3.
- **A call cannot be confirmed as a conversation without its transcript.** Duration is not a usable proxy. A message-eligible contact needs no transcript; a call-only contact cannot be confirmed without one. See Step 4.
- **Sending is a separate, approved step.** Drafts are always presented for review first. Approval covers one batch and is never standing. See Step 7.

Before drafting messages, read [seasonal-hooks.md](seasonal-hooks.md). Before writing output, read [output-template.md](output-template.md).

## Step 1 — Set the dormancy threshold

`dead_thread_days` controls the cutoff. Default to **90**.

1. Use the user's explicit threshold when given ("six months", "since spring").
2. Otherwise default to 90 days.
3. Compute `cutoff` = run time minus `dead_thread_days`, in the user's timezone.
4. State the threshold and the ISO-8601 cutoff at the top of the output.

Do not interview the user about configuration up front. A small business does not want a settings questionnaire.

Revisit the threshold only when the result looks implausible — zero eligible contacts, or more than roughly a third of everyone they have ever talked to. Then say what was used, why the count looks off, and offer a specific alternative.

Vertical changes what dormancy means. Six months between HVAC touches is a normal maintenance cycle; 90 days for a moving company is permanently gone. [seasonal-hooks.md](seasonal-hooks.md) carries per-vertical guidance — consult it before overriding.

## Step 2 — Read Quo safely

**Run every Quo fetch sequentially. Never issue parallel, concurrent, batched, or `Promise.all`-style MCP requests. One inbox and one fetch at a time.**

Call `list-inboxes` first. Respect any inbox filter the user gives; otherwise use all accessible inboxes, finishing one completely before starting the next.

Capture the teammate roster from `list-inboxes` now. It is the authoritative source for sender names (see Step 5).

### Discovery paging — read this before writing any query

Whole-inbox `fetch-messages` walks a conversation cursor ordered by recency, roughly 100 conversations per page.

**Do not pass `createdAfter` or `createdBefore` during discovery paging.** A date-filtered whole-inbox query that matches nothing returns no `next_conversations` token, which silently ends paging before the cutoff is reached. Page unfiltered and filter timestamps client-side. This is the single most likely way to get a wrong answer from this skill.

For each page:

1. Call `fetch-messages` with the inbox and, after the first page, `conversationPageToken`.
2. Record every participant with their newest and oldest timestamps in that page.
3. Interpret coverage signals exactly:
   - `[next_conversations: TOKEN]` — continue sequentially with `conversationPageToken: TOKEN`.
   - `[incomplete: N more participant(s) ...]` — there is **no** token for the omitted participants. Never treat `[incomplete]` as a paging signal. Note the shortfall and disclose it.
4. Stop paging when the newest timestamp on a page falls before `cutoff`, or at **5 pages**, whichever comes first.

`maxResults` is ignored in whole-inbox mode. Do not lower it to control volume; it has no effect.

If `[incomplete]` appears on most pages, the inbox is too busy for cursor discovery. Say so plainly, report what was covered, and recommend a Quo Analytics CSV export instead of presenting a partial list as complete. High-volume shared lines can produce several hundred participants per week, which exhausts the cursor long before a 90-day cutoff.

## Step 3 — Confirm dormancy per candidate

Page results are not proof of dormancy. The cursor is not strictly ordered by last activity, so a page reaching back two months can still contain a contact who messaged yesterday. Confirm every candidate individually.

Rank candidates before confirming, then confirm at most **25** by default. Rank by strength of prior engagement first (see Step 4), then by how recently they went quiet — a contact who went quiet four months ago is more recoverable than one who went quiet two years ago. Disclose the cap and the number of unconfirmed candidates.

For each candidate, sequentially:

1. `fetch-messages` with `participantPhoneNumber` and `maxResults: 3`. Here `maxResults` **is** honored.
2. `fetch-call-transcripts` with `participantPhoneNumber` for call-channel activity.

### Automated outbound does not count as activity

Dormancy means the *relationship* went quiet, not the thread. Measure the last activity date from **only** these:

- any inbound message or call from the contact
- any outbound message or call made by a human teammate

**Exclude every automated outbound send** from the calculation: post-call surveys, callback notices, booking-link templates, missed-call autoreplies, drip sends.

Identify automated outbound by identical or near-identical text recurring across unrelated contacts and threads. A template blasted to hundreds of people is unmistakable once two threads are compared.

Skipping this inverts the whole skill. On any line running surveys or autoreplies, the business's own automation keeps touching dormant contacts and no one ever appears dormant — so the customers who most need this get a near-empty list.

Drop the candidate only if *human* activity appears at or after `cutoff`.

An inbound missed call counts as the contact reaching out, so it is activity and disqualifies dormancy. Add a per-contact `fetch-missed-calls` check when precision matters more than call budget. Do not rely on missed calls to *identify* candidates: in multi-participant discovery the `participants` field can come back empty, and `conversationId` is not accepted as a filter by any fetch tool, so those rows cannot be resolved to a person.

## Step 4 — Classify: did they actually talk?

A contact is eligible only if at least one holds:

- **An inbound message from the contact that is not an auto-responder.** One-word replies count — "Great", "Good", "Professional and helpful" are thin as messages but each confirms a human exchange happened.
- **A call whose transcript shows a genuine two-way exchange**, judged by turn alternation. Never by duration.

Everything else is **never-connected**: they called or were called, reached a machine or a voicemail, and never engaged. Keep this group out of the main list. Report the count so the user knows the pool exists, and note that it is a different and much weaker play.

### Never use call duration as a proxy for conversation

Duration cannot distinguish a conversation from a phone tree, and the failure is not marginal. Observed in testing: five separate "completed" calls to one contact at 105s, 114s, 117s, 132s and 136s were *all* the contact's IVR menu running into voicemail, with no human ever on the line. That contact's phone tree consumed 85 seconds on menu options before the voicemail greeting began. A separate 68-second "completed" call was a voicemail drop — machine greeting, then the rep talking to nobody.

Any duration floor that admits a real short conversation also admits a long phone tree. There is no threshold that separates them.

### Detecting IVR and voicemail

**The attribution trap:** transcripts attribute IVR and voicemail-greeting audio to the *participant's* phone number. So "the participant has speech in this transcript" does not mean a person spoke. Judging on speaker labels alone will classify every phone tree as a conversation.

Use turn alternation as the primary test. A real conversation alternates speakers repeatedly — roughly three or more genuine back-and-forth switches. IVR and voicemail produce one long machine block, optionally followed by a single rep monologue, and then end.

Corroborate with machine phrases. Any of these in an early block marks it as automated:

- "Thank you for calling [business name]"
- "listen carefully to the following options", "press 1", "press 2"
- "please hold while we connect", "extension"
- "this call will be recorded for quality"
- "we are currently closed", "away from the desk", "assisting other clients"
- "please leave your full name, phone number, and a detailed message"
- "please leave your message for [number]"
- "to review the voice mail, press 1", "to send the voice mail now, press 2"

A transcript that is a machine block plus a rep monologue is a **voicemail drop**, not a conversation. Classify as never-connected.

### Excluding auto-responders in messages

Businesses text other businesses, and the other end is often a machine. Treat an inbound message as automated when it shows any of:

- Arrival within roughly 15 seconds of an outbound message. Observed in testing: one reply landed **845 milliseconds** after the outbound send.
- Identical text repeated across separate, unrelated outbound attempts.
- SMS program boilerplate — "You'll now receive communications from", "Message and data rates may apply", "Text STOP to cancel or HELP for help".
- A bare portal, booking, or maps URL with no response to what was asked.
- Away, after-hours, or missed-call autoreply phrasing such as "Sorry to miss your call" or "How can we help?" with no other content.

Two or more signals is conclusive. One is a flag — check whether any other message in the thread reads as human, and check the call transcripts, before deciding.

**A single thread can contain both a machine and a person.** Observed in testing: one contact's thread held an insurance company's SMS opt-in boilerplate and then, two seconds later, "Very good. Monica was excellent." That contact is eligible. Do not tighten this into "any automated marker disqualifies the contact" — that rule would have dropped a real conversation. Automated markers disqualify the *message*, never the contact.

Failing to strip these produces a list of vending machines, which destroys trust in the skill on first run.

### Excluding test threads and internal numbers

Auto-responder detection does not catch these, because two real humans are talking. They pass turn alternation, carry no machine phrases, and would ship a re-engagement text to the business's own staff.

Observed in testing: a thread whose entire content was an outgoing "Test" and an incoming "Check" 23 seconds later. Genuine alternation, genuine humans, and someone checking their own line.

Exclude a contact when either holds:

- **The whole exchange is trivially short with no substantive content.** Messages under roughly 15 characters, on both sides, with nothing else in the thread. "Test", "Check", "ok", "1" — setup verification, not a conversation. A short *reply* to a real question is different and stays eligible; the test is whether the entire thread is content-free.
- **The number belongs to the workspace or its teammates.** Compare against every inbox number from `list-inboxes`, captured in Step 2. Also treat a contact as internal when a transcript or thread shows the participant identifying as staff.

Do these checks before spending confirmation or transcript calls on a candidate — they are free and they remove work.

Two related cases to hold out rather than draft for, and to report as counts:

- A number appearing in more than one of the workspace's own inboxes is likely a teammate's cell or a forwarding number.
- A contact whose only substantive exchange is about setting up Quo itself, rather than about their own business, may be a staff member or a partner. Flag for the user rather than guessing.

## Step 5 — Pull transcripts

Transcripts do two jobs: they confirm call eligibility for call-only contacts (Step 4), and they supply the personalization hook.

Pull **per contact** using `participantPhoneNumber`, never in bulk. Bulk transcript fetches cap at 100 records and return **no `[incomplete]` marker, no token, and no warning at all**, so a truncated result is indistinguishable from a complete one. Per-contact pulls also give exact attribution.

### Compact after every pull

Transcripts are enormous and will exhaust context long before a candidate list is finished. Observed in testing: a single 1,114-second call returned roughly 200 turns. Fourteen of those cannot be held at once.

So process strictly one at a time:

1. Pull one contact's transcripts.
2. Extract and write down, in a few lines at most: the specific problem they raised, any objection, a competitor named, timeline or budget language, their trade or vertical, and the handling teammate.
3. **Discard the raw transcript text before pulling the next.** Keep only the compacted notes.
4. Move to the next contact.

Never hold two raw transcripts at once, and never pull the whole candidate set before extracting. Skipping compaction does not fail loudly — it quietly truncates the run partway through the list and reports thin coverage that looks like missing data rather than an avoidable mistake.

### Never take names from transcript text

Speech-to-text mangles proper nouns badly. Observed in testing: a rep named Cristine rendered as "Christine calling from Cole"; a rep named Jheyssa rendered as "Jisa" and then "Jaseh" in the same transcript; a contact named Mina rendered as "Nina".

Source names only from structured data — the `list-inboxes` roster for teammates, and Quo contact records via `get-contact` for contacts. A misspelled name in the draft is worse than no name.

### Resolving which teammate handled a call

`fetch-call-transcripts` accepts a `userId` filter but never returns a userId field, so attribution works only by probing: query the call with a teammate's ID and see whether it comes back.

This is verified. In testing, a call whose transcript garbled the rep as "Jisa" and then "Jaseh" returned under Jheyssa Alas's ID, and returned nothing under a different teammate's ID. The filter is real, not ignored, so a match is genuine attribution.

It is also expensive, because every probe re-returns the full transcript text. Use it sparingly:

1. **Only when the roster is small.** Get the roster from `list-users`. At 2–5 users this costs a few probes; on a large shared line it is not worth it. Skip attribution above roughly 5 users and draft without a rep name.
2. **Narrow to the single call.** Pass `createdAfter` and `createdBefore` bracketing the known call timestamp by a minute, so each probe returns one call rather than the teammate's whole history.
3. **Order probes by likelihood.** Try the teammate whose name is closest to the garbled transcript rendering first, then others assigned to that inbox. This usually resolves on the first probe.
4. **Compact and discard between probes**, exactly as above.
5. **Stop after the roster is exhausted or the budget is spent.** Draft without a name rather than continuing.

Exhausting the roster with no match is a real result, not an error: a call handled by an AI agent matches no user. Do not attribute such a call to a person.

Only probe when the name materially improves the draft — a contact who had a long substantive call remembers who they spoke to. For a one-word survey reply, the name adds little and is not worth the transcript cost.

### When there is no transcript

For a **message-eligible** contact, fall back to the message thread, which carries more than expected — a stated trial signup, a named integration they needed, or pasted links that reveal their industry. They stay on the list with a thinner message. Never drop a message-eligible contact for lack of a transcript.

For a **call-only** contact with no transcript, eligibility cannot be established. Do not guess from duration. Hold them out of the main list and count them separately as unverifiable.

State coverage as counts: "call context for 9 of 14 eligible". Never imply an uncovered contact did not talk.

### Plan consequence

Transcripts require the Quo Business plan with transcription enabled. On a plan or period without it, only message-eligible contacts can be surfaced, and call-only contacts are unverifiable. Say this plainly rather than returning a short list with no explanation.

## Step 6 — Draft the messages

Read [seasonal-hooks.md](seasonal-hooks.md) for vertical inference and the time-of-year map.

Each message needs four things:

1. **Named sender**, and where possible the teammate who actually handled the call — from the roster, not the transcript.
2. **The specific month** of the last conversation. Naming it is most of what separates this from a blast; "a while back" reads as bulk.
3. **One concrete detail** from the transcript or thread.
4. **A seasonal reason to be texting now** drawn from the contact's vertical and the current date — not the sender's.

Constraints:

- Keep under 320 characters so it stays two SMS segments and does not convert to MMS.
- One question maximum.
- Include an opt-out where the relationship is thin or the last contact is distant.
- No urgency or scarcity language. These are dormant relationships, and pressure reads as spam.
- Never fabricate a detail. Drop to the generic version instead.

Present every draft for review. Drafting and sending never happen in the same pass.

## Step 7 — Send, only after approval

Sends cannot be undone. Treat every instruction in this step as a guardrail on an irreversible action.

### Approval is per batch and never standing

- Present the drafts, stop, and wait for the user to approve. Never send in the same pass as drafting.
- Approval covers only the batch in front of the user. It never carries to the rest of the list and never carries to a later run.
- **Do not add a configuration option that enables sending without review.** `dead_thread_days` is configurable; approval is not. A version of this skill that can send unattended will eventually text a wrong message to someone's customer with nobody watching.
- If the user says "send them all", send the first batch only, report, then continue on their word.

### Editing drafts

Expect edits. Drafts are a starting point, and the user knows their contacts better than any inference in Step 5 does.

- Honor any edit request and re-present the changed draft in full, so the user sees exactly what will send.
- **An edit re-opens approval for that draft.** Never send an edited draft on the strength of an approval given before the edit. Approval attaches to specific wording, not to a slot in a list.
- **Send the user's wording verbatim.** Do not re-polish, re-tone, shorten, or re-apply the Step 6 constraints to text the user wrote themselves. Their phrasing is the authority; silently improving it is a betrayal of the review step.
- When a user's edit breaks a constraint — over 320 characters, no opt-out on a distant contact, more than one question — say so once, plainly, and send it as written if they confirm. Flag, never override.
- A blanket instruction ("make them all shorter", "drop the opt-out everywhere") re-opens the whole batch. Re-present every affected draft and wait again.
- Partial approval is normal and expected. "Send Mina and Dan but not the third one" is a complete instruction — follow it exactly and do not ask them to re-confirm the rest.

### Refer to drafts by contact, never by position

Position numbers shift whenever re-verification drops someone from a batch, so "send 1, 3 and 5" can resolve to different people than the user meant. Key every reference to the contact — name plus the last four digits of their number — and confirm back in those terms before sending.

If the user refers to a draft by number, restate which contact you understood before acting on it.

### Never use `send-group-message`

It places every recipient in one shared thread where each can see the others' numbers and replies. On a dormant-contact list that leaks a customer list to itself, irreversibly. Reach for it never, regardless of how many recipients are queued.

### Use personalized bulk mode

`send-bulk-messages` has two modes. Use PERSONALIZED — `messages: [{to, content}]` — because every draft differs. Never SAME-MESSAGE mode (`to` plus `content`): identical text across a re-engagement list is precisely the blast this skill exists to avoid. A number may appear only once per call. For a single recipient use `send-message`.

### Re-verify each recipient immediately before sending

Time passes between discovery and approval, and this pipeline is long. Per recipient, in the batch about to send:

1. Re-run `fetch-messages` with `participantPhoneNumber` and `maxResults: 3`.
2. Drop anyone who has made contact since discovery. Texting "we lost touch" to someone who called yesterday is the worst output this skill can produce.
3. Drop anyone who has sent STOP or any opt-out.

Report every drop with its reason. Never let the batch shrink silently.

### Batch and pause

Send at most **5** in the first batch, then stop and report. Continue only when the user says to.

This is not ceremony. The pipeline infers vertical, auto-responder status, IVR status and dormancy, and each inference carries a false-positive rate. The pause is what makes a systemic error cost five messages instead of twenty-five.

### Respect local hours

Send only during business hours in the recipient's timezone, inferred from area code where available. Hold anything outside roughly 8am–8pm local and say what was held. Some jurisdictions restrict messaging hours, and a dormant contact texted at 11pm is a complaint rather than a reply.

### Report honestly

After each batch, state what sent, what was dropped and why, and what failed. Never report a send as successful without a tool result confirming it.

## Preserve the product boundary

The Quo MCP is not a complete analytics source:

- `fetch-call-transcripts` covers completed calls with usable dialogue, not every answered call.
- Cursor discovery is bounded, capped per page, and may omit participants with no way to page to them.
- Absence from returned data does not prove an event did not occur.
- Counts describe records reviewed, not total workspace activity.

Do not calculate or claim total call volume, answer rates, conversion rates, or exhaustive contact coverage. Frame the output as "contacts found" and never as "all contacts who went quiet". When the user needs completeness, recommend a Quo Analytics CSV export and keep that workflow separate.

Say what was covered and what was not, every run.

## Consent footing

Texting people the business genuinely spoke with, from its own number, about a prior conversation, rests on an established business relationship. That is materially different from messaging cold leads, and it is why sending from this skill is reasonable at all.

That footing depends on the classification in Step 4 being right. A never-connected contact has no such relationship — which is a second reason, beyond message quality, to keep that group out of the send list entirely.

Never draft or send to a contact who has sent STOP or any equivalent opt-out, at any point in their history.

## Cross-platform behavior

Keep the skill portable between ChatGPT and Claude:

- depend only on the Quo MCP and Markdown output
- identify tools by capability when platform-specific names differ
- avoid platform-specific filesystem, code execution, memory, or UI assumptions
- preserve the same threshold, coverage caveats, classification rules, and output schema on both
