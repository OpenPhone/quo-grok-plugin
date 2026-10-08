# Output template

Use this structure. Order P1 → P2 → P3, most recent first inside each band. Render all times in the configured `timezone`.

## Empty run

Two lines. Name the mode so a glance tells you which run this was.

```
**Voicemail triage — morning** · Main Line · after hours, Fri Sep 11 5:15pm → Mon Sep 14 7:15am

No voicemails. 2 missed calls with no message left — nothing to triage.
```

## Normal run

```
**Voicemail triage — end of day** · Main Line · business day, Mon Sep 14 7:15am → 5:15pm

3 voicemails · 2 need a reply · 1 carried from yesterday · 4 tasks created

---

### Carried forward — unanswered since a previous run

**Ruth Calderon** · +1 503 ···· 1194 · left Sun 4:48pm · **17h unanswered** · [recording]
Permit question. Task from yesterday's end-of-day run is still open, no callback found.
→ **Call, then text** · escalated from P2
No new task — existing one still open on this call.

> Hi Ruth, it's Ridgeline. You left a voicemail yesterday afternoon and we were slow
> getting back — sorry. On the permit question: we handle the filing, and it adds about
> a week. Want me to start it?

---

### P1 — needs a call tonight

**Tom Vasquez** · +1 415 ···· 8821 · 4:12pm · 41s · [recording]
Duplicate charge on the September invoice, wants it resolved before the next billing
date. Called twice.
Context: A+B+C — known customer since 2023. Sept invoice discussed on a call Sep 8;
a colleague promised a credit by text on Sep 10 that hasn't appeared.
→ **Call now**
Tasks: *Pull Sept invoice for Vasquez, check duplicate charge* (due today) ·
*Call Tom Vasquez with invoice findings* (due today)

> Hi Tom, it's Ridgeline. Got your voicemail about the duplicate charge on the September
> invoice. I'm pulling it up now and will call you today with what I find.

---

### P2 — reply today

**Unverified name — heard "Marcus"** · +1 628 ···· 4409 · 2:48pm · 28s · [recording]
Asking whether someone can come Thursday. No contact record, so the name is from the
transcript and unconfirmed.
Context: A only — unknown caller, no prior thread. Tier C skipped. Gave a callback number of +1 555 ···· 0132 — **that is this
business's own second line**; the number that reaches them is the one they called from.
→ **Text** · **Needs your input** — availability slot unfilled
Task: *Confirm Thursday availability for +1 628 ···· 4409* (due tomorrow) — callback-number
warning in the task description

> Hi, it's Ridgeline — got your voicemail this afternoon about Thursday.
> «Thursday availability». Does a morning slot work?

---

### P3 — acknowledge

**Dana Okafor** · +1 917 ···· 2210 · 11:30am · 19s · [recording]
Checking in on the warranty question from last month's thread. Nothing asked for.
Context: A+B+D — warranty thread from Aug 12, coverage dates on the contact record.
Tier C skipped (P3).
→ **Text**
Task: *Send Okafor warranty paperwork* (no due date)

> Hi Dana, it's Ridgeline — got your voicemail about the warranty. It covers parts for
> two years, labour for one. Want me to email the paperwork?

---

### Partially handled — one request still open

**Alex Nwosu** · +1 312 ···· 7781 · 12:53pm · 32s · [recording]
Referral from an existing customer, wants to book a consultation, **asked for a callback**.
A colleague replied by text at 12:55pm asking for their full name — that named the
referrer, so a person clearly listened. Draft held so two of you aren't texting at once.
The callback they asked for hasn't happened, so the task stands.
→ **Call, then text**
Task: *Call Alex Nwosu back to book consultation (referral)* (due today 5pm)

Staged follow-up, **not recommended until the existing thread has had a few hours**:

> Hi, it's Ridgeline — got your voicemail about booking a consultation. I'll call you
> back today. If a particular window works better, text it to me and I'll fit around it.

---

### Just came in — held, end-of-day runs only

- **+1 971 ···· 3305** · 5:04pm · [recording] — left 11 minutes ago, someone may be
  calling back now. Task created, no draft. Surfaces next morning if still unanswered.

### Held out

- **+1 800 ···· 0147** — prerecorded business-listing pitch. No task, no draft.
- **+1 628 ···· 3264** — this business's own line. Test call or misroute, not a lead.

### Couldn't identify caller

- **Mon 9:14am** · 22s · [recording] — asking about a delivery window, no participant on
  the row and no conversation match. No number, so no draft. Recording linked for
  whoever can place the voice.

### Still transcribing

- **+1 312 ···· 5590** · 4:02pm · [recording] — still processing on re-fetch. Carried to
  the next run.

### Coverage

End-of-day run, Mon Sep 14 7:15am → 5:15pm, Main Line only. 3 voicemails in scope ·
1 carried forward from the 48h lookback · 1 held as just-came-in · 1 unidentified ·
1 still transcribing · 2 missed calls with no message · 1 held out as a pitch · 1 held
out as an internal number · 1 partially handled. 2 discovery pages walked, budget not
hit. Prior-call history pulled for 2 of 3 callers — 1 skipped as P3, none skipped for
the per-run cap. `last_run` written back as Mon Sep 14 5:15pm.

**4 tasks created. No messages sent** — drafts are for your review. Reply with which to
send, or edit any first.
```

## Section order

Mode decides what leads:

- **Morning** — Carried forward · P1 · P2 · P3 · Partially handled · Held out · Couldn't identify · Still transcribing · Coverage
- **End-of-day** — P1 · P2 · P3 · Partially handled · Just came in · Held out · Couldn't identify · Still transcribing · Coverage

Empty sections are dropped, heading and all. Carried forward never appears on an end-of-day run; Just came in never appears on a morning run.

## Formatting rules

- **Name the mode in the header** — morning, end of day, or custom, every run including empty ones.
- **Mask numbers** — country code, area code, dots, last four. Full numbers only in tool calls.
- **Name, then number.** When a name is unverified, say so on the same line. Never present a transcript-heard name as the contact's name.
- **Summarize each voicemail in two lines max**, paraphrased. Never paste the transcript.
- **Give every card a `Context:` line** naming the tiers the draft drew on (A, A+B, A+B+C+D) and, in a clause, what they contributed. State when tier C was skipped and why — P3, unknown caller, per-run cap, or transcription unavailable. This is how a reviewer knows whether a confident-sounding draft rests on the voicemail alone.
- **Flag conflicts between context and the voicemail** in the card and the task description. Never resolve one in a draft.
- **Drafts in blockquotes**, separable from commentary and easy to copy.
- **State every task created** under its caller, with due date. Tasks are the one write this skill makes on its own — never leave them implied.
- **State the `last_run` writeback** in the coverage line. If it failed, say so: the next window will double-count.
- **Say whether transcription is available.** It changes what the handled-check can see.
- **Close on the send gate.** Every non-empty run ends with the task count, an explicit statement that nothing was sent, and what to do next.
- **No answer rates, no call volumes, no percentages.** The data can't support them.
