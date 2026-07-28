---
name: documents
description: Produce the store's real documents — a GST-correct PDF invoice for a bill, and a PPTX sales-analysis deck with charts. Use when the owner asks to "send that bill as a PDF", "invoice", or "make an analysis deck / PPTX".
---

# Documents

## PDF invoice
"send me that bill as a PDF" → `generate_invoice_pdf`.
- The bill must be FINALIZED first.
- "that bill" / "the last bill" / "the invoice" with no number → call it with **no `bill_id`**; the tool
  defaults to the most recent finalized bill in this chat. Do NOT ask which bill — the owner said "that
  bill" because they mean the obvious one. Name the bill number in your reply so they can catch a mismatch.
- Only ask when they clearly mean an older bill and the reference is genuinely unresolvable.
- The shop name, GSTIN and address come from preferences (see memory skill) — no need to ask each time.

### Branding
The invoice is templated from the owner's preferences — `brand_color` (hex accent on the header band, table header and footer), `invoice_template` (`classic` light header vs `modern` full-colour band), `invoice_footer` (a line like delivery terms or a UPI handle).
- "make my bills green" / "use my shop colours" → `set_preference` key=`brand_color`.
- "show me how my invoice looks" → `preview_branding`, then offer to cut a sample.
Invoices are legal documents: they stay in English even when the owner chats in Hindi or Tamil.

## Analysis deck
"make this week's sales analysis deck" → `generate_analysis_deck` with from/to dates you compute from today.
- The deck has real charts (top items, payment mix, daily trend), a GST-slab breakup, stock-health table, a velocity-based reorder plan, an expiry watch (FEFO) and written insights. It is themed with the shop's brand colour.
- To have it arrive on its own every week, see the `scheduling` skill.

Both tools deliver the file to the owner automatically — just confirm it's on its way. Do not paste raw numbers as a substitute for the document.
