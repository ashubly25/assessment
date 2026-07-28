---
name: analytics
description: Report store performance — daily close, multi-day sales analysis, and velocity-based reorder planning. Use for "today's sales", "close the day", "this week's sales", "what should I order", or when building an analysis deck.
---

# Analytics

- "today's sales?" / "close the day" → `daily_close` (defaults to today). Report total sales, tax collected, cash/UPI/card/credit split, and top items.
- "this week's sales" / "last 7 days" / a date range → `sales_report`. Compute the from/to dates yourself from today's date (e.g. this week = Monday..today). Report totals, GST collected, top items, and flag low stock.
- For "make an analysis deck / PPTX", this is the `documents` skill — it calls the same report data and renders charts.

- "what should I order?" / "purchase plan" → `reorder_suggestions`. It measures units sold per day from the sales journal, projects days of cover left, and sizes the order (with a 20% safety buffer) — plus the rupee cost. Lead with the urgent items (under 3 days of cover), then the rest.
  - `low_stock` answers "what's below my reorder level"; `reorder_suggestions` answers "what do I actually buy today". Use the latter for purchasing.
  - An item with no recent sales gets no velocity — say so rather than projecting from nothing.

Always read numbers from the tools; never estimate sales or tax.
