---
name: inventory
description: Manage the kirana store's stock and catalogue — receive incoming goods (with batch/expiry), add new products, correct stock after breakage/theft/stock-take, and check what's left, what's expiring and what to reorder. Use for stock-in, new SKUs, stock queries, stock corrections and expiry/FEFO.
---

# Inventory

## Receiving stock
`50 packets of Maggi came in, cost ₹12, MRP ₹14`
1. `find_product` to resolve the item. If it doesn't exist, it's a new product → use `add_product`.
2. `receive_stock` with the quantity. Pass cost_price / mrp / sell_price only if the owner stated new values.
3. Confirm the new on-hand quantity.

## Adding a new product
`new item: Amul Butter 100g, GST 5%, MRP ₹62`
- Call `add_product`. If GST rate, HSN, cost, or sell price are missing, ASK — do not guess a tax slab or price.
- Slabs are **0 / 5 / 18 / 40** under GST 2.0 (in force 22 Sep 2025). The **12% and 28% slabs no longer exist** — if the owner quotes 12%, they are working from an old rate card; say so and use 5%.
  - **0%** — loose/unbranded staples, salt, fresh milk, bread, paneer, chapati
  - **5%** — everything else in a normal kirana: pre-packaged staples, edible oil, sugar, tea, butter/ghee, namkeen, biscuits, noodles, chocolate, soap, shampoo, toothpaste
  - **18%** — washing preparations/detergents (HSN 3402), most non-food household chemicals
  - **40%** — demerit only: aerated drinks, pan masala, tobacco
  Confirm with the owner if unsure.

## Queries
- "how much sugar is left?" → `find_product` then report qty (or `get_stock`).
- "what's running out?" → `low_stock` (static reorder level).
- "what should I order / purchase list?" → `reorder_suggestions` (sales velocity — says how much to buy and what it costs). Prefer this when the owner is planning a purchase.

## Batches & expiry (FEFO)
Stock lives in batches; sales consume **First-Expiry-First-Out** automatically at finalize. You never choose batches.
- Receiving goods: record them **first**, don't interrogate. Pass `expiry` (YYYY-MM-DD) and `batch_no` when the owner states them.
- Only for SHORT-shelf-life items (milk, curd, paneer, bread, eggs) is a missing date worth one short question. Packaged goods with months of shelf life (noodles, biscuits, namkeen, detergent) go in without an expiry — say it's recorded and that a date can be added later.
- "what's expiring?" / "anything going bad?" → `expiring_soon`. Flag anything already expired first.
- Expired stock is not sellable and the tools will refuse to bill it. Offer `write_off_expired`, and **confirm before writing off** — it destroys stock value. It refuses batches that aren't actually past date.
- A sale refused for stock reasons may still have expired quantity sitting behind it; say so plainly: "only 3 sellable, 6 expired — write those off?"

Never invent stock numbers or prices — always read them from the tools.

## Correcting stock (breakage, theft, stock-take)
`adjust_stock` is for stock that changed with no sale and no expiry: bottles broke, a packet was
stolen, the shelf count doesn't match the book.
- Signed `delta` in the product's own unit — negative to reduce, positive for stock found.
- A **reason is required** and goes on the audit trail. Ask for it in the owner's own words.
- **Confirm before reducing stock.** It cannot drive stock negative; if the owner's number would, the
  tool refuses and reports what is actually on hand — relay that instead of guessing at a smaller figure.
- For a stock-take ("counted 48, book says 50"), work out the delta yourself: -2. Don't ask the owner for it.
- Adjustments are journalled as `adjust`, so they never masquerade as demand in `reorder_suggestions`.
- **Expired** stock is NOT this tool — use `write_off_expired`, which refuses batches that aren't past date.
