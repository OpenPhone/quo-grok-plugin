# Output template

Use this structure and omit empty priority sections.

```markdown
# Quo conversation insight log

**Window:** <human-readable since> through <human-readable end> (<timezone>)  
**Exact boundaries:** `<createdAfter>` → `<createdBefore>`  
**Coverage:** <inboxes reviewed; messages, transcribed calls, and missed calls/voicemails; pagination or cap disclosure>

## P1 — act now

### <Contact or topic> — <short outcome>
- **When/channel:** <local datetime> · message/transcribed call/missed call/voicemail · <inbox>
- **Why it matters:** <deadline, consequence, risk, or blocker>
- **What happened:** <one or two evidence-based sentences>
- **Next step:** <actual commitment or clearly labeled recommendation>
- **Source:** <Quo identifier if returned>

## P2 — follow up

<same entry structure>

## P3 — keep in view

- **<Contact or topic>:** <one-sentence signal> · <date/channel> · <source identifier if returned>

## Patterns

- **<Pattern>:** <evidence from at least two independent conversations and why it may matter>

## Coverage limits

- This review covers returned messages, completed calls with available transcripts, and missed calls/voicemails when that tool was available inside the stated window.
- It is not a complete analytics report and does not include every answered call without a transcript.
- <Any result cap, pagination stop, inaccessible inbox, or other run-specific limitation>
```

Keep the default report to roughly ten items. Include all P1 entries, then the strongest P2 and P3 entries. If more meaningful items remain, state how many were omitted and offer a deeper log by inbox, owner, or contact.

Do not expose full phone numbers unless the user explicitly needs them. Do not quote more transcript text than needed to support the summary.
