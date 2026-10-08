# Output Template

Keep the whole result skimmable. The user should be able to read the top block, scan the table, and start approving drafts.

Order: run scope, then the ranked list with drafts, then coverage, then the never-connected count.

## Run scope

Open with four lines, no preamble:

```text
Dormancy threshold: 90 days (cutoff 2026-06-10T00:00:00-04:00)
Inbox: Front Desk (+1 555 555 5555)
Reviewed: 3 discovery pages, 41 participants, 25 confirmed individually
Eligible: 14 talked-to contacts now dormant
```

State the threshold as a number of days and an absolute cutoff. Relative phrasing alone ("last three months") is ambiguous on a re-run.

## Ranked contacts with drafts

One block per contact, strongest prior engagement first. Use the contact's name when Quo returned one; otherwise a masked number.

```text
### 1. Mina R. (…8284) — last talked Aug 25 (3.5 months)

Vertical: home services (inferred from dispatch language)
Prior engagement: two-way call with Cristine (alternating exchange
  confirmed in transcript), plus inbound message
Context: signed up for the 7-day trial, said she was likely to go with
  Quo, had open questions she wanted answered by phone
Signal strength: strong — long call, substantive inbound, stated intent

Draft:
Hi Mina, it's Cristine at Quo — we talked back in August when you
started the trial and still had a few questions open. Fall is usually
when the calls pick back up. Want me to answer those now? Reply STOP
to opt out.
```

Fields:

- **Last talked** — date plus elapsed time. Elapsed time is what drives the decision.
- **Vertical** — with the inference basis in parentheses, so a wrong guess is visible and correctable.
- **Prior engagement** — the concrete evidence of a real conversation. Name the teammate.
- **Context** — the personalization hook. Write `no call context — message thread only` or `no context available` when thin. Never leave it blank and never invent it.
- **Signal strength** — strong, moderate, or thin. Base it on engagement evidence, not on how promising the contact seems.
- **Draft** — final copy, ready to send. No placeholders. If a field could not be filled, the draft must already be rewritten to not need it.

Drafts must be complete. A draft containing `[First]` or `[Month]` is not finished work and pushes the merge back onto the user.

## Coverage

A short, plain block. Never bury a gap.

```text
Coverage
- 3 discovery pages before reaching the cutoff; cursor not exhausted
- 2 pages reported omitted participants (breadth cap) — some dormant
  contacts are likely missing from this list
- 25 of 41 candidates confirmed individually; 16 unconfirmed
- Call context available for 9 of 14 eligible contacts
```

Required whenever true:

- discovery stopped at the page cap rather than reaching the cutoff
- any page reported omitted participants
- candidates went unconfirmed because of the confirmation cap
- transcript coverage is partial
- call-only candidates could not be verified for lack of a transcript —
  report the count, and never imply they did not talk
- transcription is unavailable on the plan, so only message-eligible
  contacts could be surfaced at all

Phrase transcript gaps as missing context, never as missing conversation.

## Never-connected

One line, at the end:

```text
Also found: 63 contacts with no confirmed conversation — reached by
voicemail or template only, never engaged. Different and weaker play;
not included above.
```

Report the count so the user knows the pool exists. Do not list them and do not draft for them unless asked.

## Excluded

One line when any applied, so a suspiciously short list is explainable:

```text
Excluded: 12 auto-responders, 2 test threads, 1 internal number.
```

Add a named line for anything held out for judgement rather than filtered — a contact whose only exchange was about setting up Quo, or a number appearing in more than one workspace inbox. Those are for the user to rule on, not to silently drop.

## Tone

Write like a colleague handing over a work-in-progress, not a report generator. No executive summary, no restating the request, no closing offer to help further. The drafts are the deliverable.
