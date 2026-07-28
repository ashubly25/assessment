---
name: memory
description: Remember and apply the owner's standing preferences — default payment mode, preferred brands, and shop identity (name/GSTIN/address) for invoices. Use when the owner says "always…", "default…", "from now on…", "remember…", or sets shop details.
---

# Memory (owner preferences)

Preferences live in the database, keyed to this chat, and PERSIST across `/new` — they are not part of the conversation window.

## Setting
- "always assume UPI unless I say cash" → `set_preference` key=`default_payment` value=`upi`.
- "default atta = Aashirvaad 5kg" → `set_preference` key=`default_atta` value=`Aashirvaad Atta 5kg`.
- Shop identity for invoices → `shop_name`, `gstin`, `shop_address`, `shop_phone`, `shop_state`.
- Invoice branding → `brand_color` (e.g. `#0F766E`), `invoice_template` (`classic` | `modern`), `invoice_footer`.
- "reply in Hindi" / "தமிழில் பேசு" → `language` (`english` | `hindi` | `tamil` | `hinglish`).
- "chase khata after a week" → `khata_reminder_days`.
- Confirm what you stored.

## Applying (do this automatically, every relevant turn)
- When cutting a bill and the owner didn't state a payment mode, use `default_payment` if set.
- When the owner says just "atta" (or another item that has a stored default) and it's otherwise ambiguous, prefer `default_atta` instead of asking.
- Invoice shop header and deck theme always use the stored shop identity and branding.
- Reply in the stored `language` without being reminded — it survives `/new` like every other preference.

The preferences are also injected into your system context each turn, so you usually already know them — but use `get_preferences` if you need to double-check.
