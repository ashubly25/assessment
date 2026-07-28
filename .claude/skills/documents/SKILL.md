---
name: documents
description: Produce the store's real documents — a GST-correct PDF tax invoice for any bill, and a PPTX sales-analysis deck with real charts (sales, top items, stock health, GST collected). Use when the owner asks to "send that bill as a PDF", "invoice", "make an analysis deck / PPTX", or to change invoice branding.
---

# Documents

The store's two real deliverables. Reading this skill is what unlocks `generate_invoice_pdf`,
`generate_analysis_deck`, `set_shop_logo` and `preview_branding` — without it they don't exist for
you, so open this skill before promising the owner a document.

Both tools **write a real file and queue it for delivery**. Your reply confirms it's on its way; it
never substitutes for it. Never say a document was sent unless the tool call succeeded.

---

## PDF tax invoice

**Any finalized bill can be produced as an invoice.** `generate_invoice_pdf`, and nothing else,
builds it — `pdfkit`, one page, GST-compliant.

"send me that bill as a PDF" → `generate_invoice_pdf`.
- The bill must be FINALIZED. A draft has no invoice number and no legal standing; finalize first
  (see the `billing` skill), then generate — in the same turn, don't make the owner ask twice.
- "that bill" / "the last bill" / "the invoice" with no number → call it with **no `bill_id`**; the
  tool defaults to the most recent finalized bill in this chat. Do NOT ask which bill — the owner
  said "that bill" because they mean the obvious one. Name the resolved bill number in your reply so
  a mismatch is catchable.
- Only ask when they clearly mean an older bill and the reference is genuinely unresolvable.
- A bill belongs to the chat that cut it; the tool refuses another chat's bill. Relay that plainly.

### What the document contains
You don't assemble any of this — the tool does, from the stored bill. Know it so you can answer
questions about the invoice without opening it:
- **Header** — shop name, GSTIN, address, phone, state (all from preferences), "TAX INVOICE",
  invoice number `INV-00001` style, and the date.
- **Bill to** — customer name if the bill has one, else "Walk-in Customer" — and the payment mode.
- **One row per line item** — `#`, item, **HSN code**, qty with unit, rate, taxable value, GST%,
  **CGST**, **SGST**, amount.
- **GST summary per slab** — e.g. `5% on Rs. 459.99 = CGST Rs. 11.51 + SGST Rs. 11.50`. Odd paise
  are absorbed so the halves always sum to the tax.
- **Totals** — taxable value, CGST, SGST, grand total, plus a round-off line when rounding applies.
- **Amount in words**, and the owner's footer line.
- Text is ASCII-sanitised (`Rs.` not `₹`) because the PDF font has no rupee glyph. That is
  deliberate, not a bug — say so if the owner asks.

### Branding
Templated from the owner's preferences: `brand_color` (header band, table header, footer accent),
`invoice_template` (`classic` light header vs `modern` full-colour band), `invoice_footer` (delivery
terms, a UPI handle), and a logo.
- "make my bills green" / "use my shop colours" → `set_preference` key=`brand_color`.
- "show me how my invoice looks" → `preview_branding`, then offer to cut a sample.
- Invoices are legal documents: **always English**, even when the owner chats in Hindi or Tamil.

---

## PPTX analysis deck

**On request the agent builds a real PowerPoint analysing the store.** `generate_analysis_deck`
does it with `pptxgenjs`; the charts are native PowerPoint chart objects with embedded data
series, not images of charts, so the owner can click into them.

"make this week's sales analysis deck" → `generate_analysis_deck` with from/to dates you compute
from today. "this week" = Monday to today; "last month" = that calendar month. State the range you
used in your reply.

### What the deck contains
Up to 6 slides, themed with the shop's brand colour and logo. Slides with nothing to say are
dropped rather than padded:
1. **Title + KPIs + insights** — total sales, bill count, **GST collected**, average bill, khata
   outstanding, and written takeaways (best seller, dominant payment mode, GST liability, cash to
   collect, expired stock still on the shelf).
2. **Top selling items & payment mix** — bar chart of revenue by item, doughnut of payment split.
3. **Daily sales trend & GST breakup** — line chart by day, plus a table of taxable and tax per slab.
4. **Stock health** — what is at or below reorder level, or an explicit all-clear.
5. **Reorder plan** — velocity-based: units/day, days of cover, suggested order and its rupee cost.
6. **Expiry watch (FEFO)** — batches nearest expiry, days left, value at risk, expired flagged first.

The figures come from `sales_report` / `reorder_suggestions` / `expiring_soon` in the `analytics`
and `inventory` skills — the same numbers the owner gets from "today's sales", never recomputed by
hand in prose.

To have the deck arrive on its own every week, see the `scheduling` skill.

---

## Rules
- One tool call produces one document. Don't generate twice to "make sure" — the owner gets two files.
- If a tool returns an error (draft not finalized, wrong chat, no bills in range), relay it and say
  what would fix it. Never paste a hand-made table as a stand-in for the document.
- Don't quote figures you didn't get from a tool. The document is the artifact; your message is a
  covering note.
