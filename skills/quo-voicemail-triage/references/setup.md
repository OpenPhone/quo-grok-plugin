# First-run setup

Run this when there is no `Voicemail Triage — settings` task. It is a conversation, not a form: ask, wait, react. Five questions, then a calibration run with no writes.

Open by saying what the skill does and what it will never do, in two sentences. People need to know before they answer anything that it creates tasks on its own and never sends a text without them.

## 1. Which line?

Call `list-inboxes` and show the numbers with their names and assigned users. Ask which one receives voicemails from customers.

- **One inbox** — confirm it rather than assuming: "Looks like there's just the one, [number]. Is that the line customers call?"
- **Several** — let them pick. If they say "all of them", explain that settings are per line (hours and templates differ) and offer to set up the busiest one now and repeat for others later.
- **They pick a line that makes mostly outbound calls** — say so plainly. On outbound-heavy lines, callers hang up rather than leave messages and most runs will be empty. Ask whether a different line takes more inbound.

Stop here for an answer. The rest reads differently once the line is known.

## 2. When are you open?

Ask for business days and hours in one question: "What days and hours is the line staffed?"

Accept vague answers and firm them up: "weekdays 9 to 5", "Mon-Sat, 8 till 6", "we're 24/7". Confirm the timezone explicitly — never infer it from an area code, because plenty of businesses keep a number from a previous city.

Then state the two run times you are deriving, so they can push back:

- **Morning run** — 45 minutes before open. Early enough to read before the phones start.
- **End-of-day run** — 15 minutes after close.

For a line staffed around the clock, there is no "closed stretch". Offer two evenly spaced runs instead — roughly start of shift and mid-shift — and say why.

## 3. Who picks these up?

Call `list-users`. Ask who should be assigned the follow-up tasks by default.

Offer *unassigned* as a real choice, not a fallback — plenty of small teams would rather triage a shared list than have tasks pre-routed. Say what each means: assigned tasks land in one person's queue; unassigned ones sit in the inbox for whoever gets there first.

## 4. Any numbers to ignore?

Ask whether they have test lines, demo numbers, or other business numbers that call this line. `list-inboxes` catches lines in the same workspace automatically, but not another workspace or a personal mobile used for testing.

This is worth asking even when the answer is no. Without it, a test call becomes a task, and on an unattended run nobody is there to notice.

## 5. Discover the line's automated texts

Do not ask them to describe their templates — people forget the exact wording, and exact wording is what the check needs.

Instead: whole-inbox `fetch-messages` over the last 14 days. Look for outbound texts appearing **verbatim across three or more unrelated contacts**. Those are templates. Also flag any outbound carrying an opt-out footer.

Show what you found, trimmed to a recognizable first line each, and ask them to confirm which are automated. Store confirmed fingerprints in `templates`.

Explain why in one sentence: without it, an auto-response sent after a voicemail looks like a colleague already handled the caller, and the voicemail gets silently dropped.

If nothing repeats, say so — some lines genuinely have no automation — and note that the general signals in Step 3 still apply.

## 6. Calibration run — no writes

Run the full triage over the **last 7 days**, or 14 if 7 is empty. Create nothing and send nothing. Present it exactly as a real run would look, with one line at the top making clear that no tasks were created and no messages sent.

This is the most valuable part of setup. It is where someone sees that the skill called a pushy sales voicemail P3, or banded a billing complaint as P1, and can correct the judgment before it touches anything.

Ask two questions afterward:

1. Did anything land in the wrong band?
2. Would you have sent these texts?

Fold real corrections into the settings — a vertical-specific urgency cue, a caller type to always hold out, a phrase never to use. Record them as `notes` in the settings task and honor them on every run.

If the calibration window is empty, do not present an empty report as a successful setup. Say the line had no voicemails in that stretch, that the skill is configured and will run, and that there is nothing yet to check the judgment against.

## 7. Write the settings task

`create-task`, titled `Voicemail Triage — settings`, linked to the inbox, with every field in the description one per line, including `last_run` set to the moment setup finished.

Read the task back and confirm the write landed. Then tell them the two run times in their own timezone, and that tasks appear on their own while texts always wait.

## If setup is interrupted

Never write a partial settings task — a half-configured run is worse than an unconfigured one, because it looks like it worked. If they break off partway, say which answers you have and that setup will resume there. Nothing durable is written until step 7.
