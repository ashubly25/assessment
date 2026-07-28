---
name: scheduling
description: Set up recurring things the bot sends on its own — the weekly analysis deck, khata payment reminders, and the end-of-day summary. Use when the owner says "every Monday", "each week", "every evening", "remind me automatically", or asks what's scheduled.
---

# Scheduling (automatic pushes)

The bot has its own clock. Once a schedule is set, it sends without being asked — the owner does not need to be in the chat.

## Kinds
| Kind | What arrives | Typical ask |
|---|---|---|
| `weekly_deck` | PPTX analysis of the last 7 days, plus a one-line summary | "send me the sales deck every Monday morning" |
| `khata_reminder` | Digest of customers who owe and have gone quiet | "remind me about udhaar every Friday" |
| `daily_close` | End-of-day totals, payment split, top items | "send me the day close every night at 10" |

## Setting one
`set_schedule` with `kind`, `weekday` (0=Sunday … 6=Saturday; omit for every day) and `hour`/`minute`.
- Map the owner's words to a time: "Monday morning" → weekday 1, hour 9. If they said a day but no time, pick a sensible hour and **tell them which** — don't interrogate.
- Setting the same kind again replaces the old one. Say what changed.
- Times are the shop server's local time.

## Managing
- "what's scheduled?" → `list_schedules`.
- "stop the weekly deck" → `cancel_schedule`.
- "show me what it'll look like" → `run_scheduled_job_now` — a dry run that delivers immediately without disturbing the schedule.

## Rules
- A job claims its slot atomically, so a restart or a slow run never double-sends. Don't build your own retry.
- Never promise a push that isn't actually scheduled — confirm with the tool result, and quote the day/time back.
- The khata digest goes to the OWNER. The bot never messages customers directly; for that, use `draft_khata_reminder` in the khata skill.
