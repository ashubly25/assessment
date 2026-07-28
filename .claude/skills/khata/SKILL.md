---
name: khata
description: Run the customer credit ledger (khata) — charges, payments, balances, and reminders for customers who have gone quiet. Use for any credit/udhaar/khata request.
---

# Khata (credit ledger)

Khata is informal customer credit — a core kirana concept. Balance is what the customer owes the shop.

- "put ₹500 on Ramesh's credit" → `khata_charge` (customer=Ramesh, amount=500). Creates Ramesh if new.
- "Ramesh paid ₹300" → `khata_payment`. This is REFUSED if Ramesh has no khata or pays more than owed — relay the refusal and confirm the correct amount.
- "Ramesh's balance?" / "how much does Ramesh owe?" → `khata_balance`.

Notes
- A bill paid by `credit` books its total to the customer's khata automatically on finalize (see billing skill) — don't double-charge.
- Always confirm the new balance after a charge or payment.
- Never invent a balance — read it from `khata_balance`.

## Reminders
- "who owes me?" / "pending udhaar" / "khata reminders" → `khata_reminders` (default: no payment in 14+ days). Report names, amounts and how long outstanding.
- "remind Ramesh" → `draft_khata_reminder`. It returns a polite message the owner forwards over WhatsApp/SMS — the bot does not message customers itself. Say so; don't imply it was sent.
- To have this pushed automatically, use the `scheduling` skill (`khata_reminder`). Owner can tune staleness with the `khata_reminder_days` preference.
