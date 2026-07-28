---
name: inventory
description: Manage the kirana store's stock and catalogue — receive incoming goods (with batch/expiry), add new products, check what's left, what's expiring, what to reorder, and link barcodes. Use for stock-in, new SKUs, stock queries, expiry/FEFO and barcode scans.
---

# Inventory

## Receiving stock
`50 packets of Maggi came in, cost ₹12, MRP ₹14`
1. `find_product` to resolve the item. If it doesn't exist, it's a new product → use `add_product`.
2. `receive_stock` with the quantity. Pass cost_price / mrp / sell_price only if the owner stated new values.
3. Confirm the new on-hand quantity.

## Adding a new product
`new item: Amul Butter 100g, GST 12%, MRP ₹62`
- Call `add_product`. If GST rate, HSN, cost, or sell price are missing, ASK — do not guess a tax slab or price.
- Typical Indian slabs: loose/unbranded staples & salt & milk & bread = 0%; packaged staples, edible oil, sugar, tea = 5%; butter/ghee, namkeen = 12%; biscuits, noodles, detergent, soap, chocolate, toothpaste = 18%. Confirm with the owner if unsure.

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

## Barcodes & photos
- Owner sends a barcode photo → read the digits printed under the bars → `find_by_barcode`.
- No match → `find_product` on the brand name visible on the pack, confirm the SKU with the owner, then `set_barcode` so the next scan is instant.
- Owner sends a product photo with no barcode → identify brand + pack size, then confirm via `find_product`. Never bill straight from a photo without the owner confirming the SKU.

Never invent stock numbers or prices — always read them from the tools.
