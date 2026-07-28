---
name: vision
description: Handle photos the owner sends — barcode scans, product-pack shots, and the shop logo. Use whenever a message includes an image. Read the picture yourself and act; there is no OCR/classifier branch.
---

# Vision (photos)

A photo arrives as an image you can see directly in the turn, usually with a short caption (or none). Decide what it is and route it — never guess a SKU or price from a picture alone.

## Barcode photo
1. Read the **digits printed under the bars** (do not try to decode the bars visually) → `find_by_barcode`.
2. No match → `find_product` on the brand/size text visible on the pack, confirm the SKU with the owner, then `set_barcode` so the next scan of that item is instant.
3. If the caption also states an action ("bill this", "10 came in"), do it **only after** the SKU is confirmed — a photo is never enough on its own to bill or decrement stock.

## Product-pack photo (no readable barcode)
- Identify brand + pack size from the label, then confirm via `find_product`. If several sizes match (e.g. atta 5kg vs loose), ask which — same disambiguation rule as text.
- Then continue whatever the owner asked (receive stock, add to a bill, add a new product).

## Shop logo / signboard
- Treat a photo as the logo **only when the owner says so** ("make this my logo", "put this on my bills"). Then call `set_shop_logo` — it uses the image just sent and applies it to invoices and decks.
- Don't assume a random product photo is a logo.

## Rules
- Confirm the matched SKU (name + size + price from the tool) before billing or stock changes — state what you saw and what you matched.
- If the image is blurry/ambiguous, say what you can and can't read and ask one short question rather than guessing.
- Photos are grounding *hints*, not sources of truth — prices, GST and stock still come only from the tools.
