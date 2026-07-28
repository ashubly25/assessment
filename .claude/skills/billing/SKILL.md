---
name: billing
description: Cut, edit and reverse customer bills for the kirana store — resolve items, build a multi-turn draft, apply GST, take payment, finalize, and void a finalized bill that was wrong. Use whenever the owner wants to make/edit a bill, sell items, check the current bill, or undo a completed sale.
---

# Billing

Cut a GST-correct bill from terse owner messages like
`make a bill: 2kg sugar, 1 aashirvaad atta, 4 maggi, 1 amul butter, UPI`.

## Flow
0. `start_bill` opens a draft for this chat. `add_bill_item` auto-opens one, so call `start_bill`
   explicitly only when the owner clearly begins a fresh bill and you want it on the record; it never
   opens a second draft alongside an open one. `show_bill` prints the current draft with its GST
   breakup — use it when the owner asks "what's on the bill?" or after several edits.
1. For EACH item, call `find_product` to get the real product id, price and GST. Never assume a price.
   - If a term is ambiguous (e.g. "atta" → Aashirvaad 5kg vs loose wheat atta), ASK which one before adding it. Add the rest meanwhile if unambiguous.
   - Quantities: loose items are in kg/g/litre (e.g. "2kg sugar" → qty 2). Packaged items are counts ("4 maggi" → qty 4).
2. Call `add_bill_item` for each resolved item. This builds a DRAFT; stock is not touched yet. It enforces the oversell guard and refuses below-cost sales — if it returns an error, tell the owner plainly (e.g. "only 6 Maggi in stock") and ask how to proceed.
3. Edits mid-build ("drop the butter", "make it 6 Maggi") → `set_bill_item` (qty 0 removes). Re-show the running total.
4. Payment: use `set_bill_meta` with cash/upi/card/credit. If the owner didn't say and has a default payment preference, apply it. For UPI/card, capture a reference if given. Credit REQUIRES a customer name (books to khata).
5. `finalize_bill` — only now is stock decremented, atomically. It is idempotent; never call it twice to "make sure".
   - A mode named while items are still being listed ("2kg sugar, 4 maggi, UPI") is **recorded, not a
     trigger**: keep the draft, show the total, say it's ready. The very next message is often "drop the
     butter, make it 6 Maggi", and after finalizing that correction costs a void of a completed sale.
   - Finalize when the owner closes the sale: "finalize", "done", "that's all", "settle it".
   - **If you asked for the payment mode and the owner replies with just the mode ("cash", "UPI"), that
     answer IS the close** — set it and finalize in the same turn. Do not turn around and ask "finalize
     now?"; they answered the only question left. This is the opposite case from a mode buried in an item
     list, where the bill has not been shown yet and a correction is still likely.

## Rules
- Show the bill with its per-slab GST breakup and total after meaningful changes and before finalizing.
- Do not finalize without a payment mode.
- If the owner asks for the invoice as a PDF, that is the `documents` skill — finalize first, then generate.
- Anything that needs a finalized bill (invoice PDF, today's sales) while a draft is open with a known
  payment mode: finalize it first, then do what they asked — in one turn, not two.
- Prices, GST rates and stock come **only** from `find_product` / `get_stock`. Never invent a product,
  a price or a stock number; if unsure, look it up again.
- The tools own the hard rules — oversell guard, GST maths, below-cost refusal, idempotency. When a tool
  returns an error, relay it plainly and suggest the fix. Never try to work around a guard.
- Money is ₹ (INR). Be terse and shopkeeper-friendly, and confirm with the key numbers.

## Undoing a finalized bill
A finalized bill is a completed sale — stock has moved and, if it was credit, a khata was charged.
`void_bill` is the only way back, and it is not an edit:
- **Always confirm first.** Say what will be reversed (bill number, total, stock returned, khata unwound) and wait for a yes. Never void on your own initiative.
- **One exception:** the owner is plainly correcting a bill you just finalized ("drop the butter, make it
  6 Maggi"). That is their instruction, not your initiative — void and rebill in the same turn, then
  report the void and the new bill number. Asking again just strands the correction, and repeating the
  question on later turns is worse.
- A **reason is required** and is recorded on the bill. Ask for it if the owner didn't give one ("wrong customer", "duplicate entry", "customer returned everything").
- It restores the exact batches the sale consumed, so expiry dates stay honest, and it reverses the khata charge for a credit sale.
- The bill and its lines are **kept and marked void** — they stay in the audit trail. Nothing is deleted, and a voided bill is excluded from sales figures.
- If some stock can't be returned (its batch was written off since), the tool says so — pass that on rather than hiding it.
- A **draft** is not voided; just edit it with `set_bill_item`, or leave it.
- Partial return of one item? Void the bill and cut a corrected one. There is no partial-refund tool, and inventing one in prose is worse than saying so.
