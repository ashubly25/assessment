---
name: billing
description: Cut, edit and reverse customer bills for the kirana store — resolve items, build a multi-turn draft, apply GST, take payment, finalize, and void a finalized bill that was wrong. Use whenever the owner wants to make/edit a bill, sell items, check the current bill, or undo a completed sale.
---

# Billing

Cut a GST-correct bill from terse owner messages like
`make a bill: 2kg sugar, 1 aashirvaad atta, 4 maggi, 1 amul butter, UPI`.

## Flow
1. For EACH item, call `find_product` to get the real product id, price and GST. Never assume a price.
   - If a term is ambiguous (e.g. "atta" → Aashirvaad 5kg vs loose wheat atta), ASK which one before adding it. Add the rest meanwhile if unambiguous.
   - Quantities: loose items are in kg/g/litre (e.g. "2kg sugar" → qty 2). Packaged items are counts ("4 maggi" → qty 4).
2. Call `add_bill_item` for each resolved item. This builds a DRAFT; stock is not touched yet. It enforces the oversell guard and refuses below-cost sales — if it returns an error, tell the owner plainly (e.g. "only 6 Maggi in stock") and ask how to proceed.
3. Edits mid-build ("drop the butter", "make it 6 Maggi") → `set_bill_item` (qty 0 removes). Re-show the running total.
4. Payment: use `set_bill_meta` with cash/upi/card/credit. If the owner didn't say and has a default payment preference, apply it. For UPI/card, capture a reference if given. Credit REQUIRES a customer name (books to khata).
5. `finalize_bill` — only now is stock decremented, atomically. It is idempotent; never call it twice to "make sure".

## Rules
- Show the bill with its per-slab GST breakup and total after meaningful changes and before finalizing.
- Do not finalize without a payment mode.
- If the owner asks for the invoice as a PDF, that is the `documents` skill — finalize first, then generate.

## Undoing a finalized bill
A finalized bill is a completed sale — stock has moved and, if it was credit, a khata was charged.
`void_bill` is the only way back, and it is not an edit:
- **Always confirm first.** Say what will be reversed (bill number, total, stock returned, khata unwound) and wait for a yes. Never void on your own initiative.
- A **reason is required** and is recorded on the bill. Ask for it if the owner didn't give one ("wrong customer", "duplicate entry", "customer returned everything").
- It restores the exact batches the sale consumed, so expiry dates stay honest, and it reverses the khata charge for a credit sale.
- The bill and its lines are **kept and marked void** — they stay in the audit trail. Nothing is deleted, and a voided bill is excluded from sales figures.
- If some stock can't be returned (its batch was written off since), the tool says so — pass that on rather than hiding it.
- A **draft** is not voided; just edit it with `set_bill_item`, or leave it.
- Partial return of one item? Void the bill and cut a corrected one. There is no partial-refund tool, and inventing one in prose is worse than saying so.
