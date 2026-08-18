# DOCKLINE — Smart Warehouse Operations & Order Fulfillment System

A single-file, no-install web app built for the *Smart Warehouse Operations & Order
Fulfillment System* hackathon problem statement. It simulates a real warehouse
control tower: inventory, orders, picking/packing, exceptions, and analytics —
all driven by decision logic, not just data display.

## How to run

1. Unzip this package.
2. Open **`index.html`** in any modern browser (Chrome, Edge, Safari 16.2+, Firefox 128+).
   No build step, no server, no install required — it loads React, Chart.js and
   fonts from public CDNs and runs entirely client-side with mock in-memory data.
3. Optional: serve it locally instead of opening the file directly —
   `python3 -m http.server 8000` from this folder, then visit `http://localhost:8000`.

## What it covers (mapped to the problem statement)

| Requirement | Where it lives |
|---|---|
| Inventory & stock monitoring | **Inventory** tab — stock, reserved, damaged, available, reorder point per SKU with bin location tags |
| Order management & prioritization | **Orders** tab + **Priority Conveyor** on the dashboard — every order gets a computed priority score |
| Inventory allocation | Allocation engine (`runAllocation`) — greedy, priority-ranked, cross-order allocation with a live decision log |
| Picking & packing management | **Picking & Packing** tab — pick lists auto-sequenced by zone → aisle → bin to minimize travel |
| Low-stock & out-of-stock detection | Reorder engine (`reorderRecommendation`) — lead-time-aware reorder quantity, surfaced on Dashboard + Inventory |
| Damaged/missing item handling | **Exceptions** tab — report from the floor, system proposes a resolution and re-triggers allocation |
| Fulfillment & dispatch tracking | Order stage tracker: Created → Allocated → Picking → Packing → QC → Dispatched |
| Operational analytics & bottlenecks | **Analytics** tab — pipeline stage distribution, stage-duration bottleneck chart, stock-vs-reorder chart |

## The decision engine (the "competitive twist")

This isn't a CRUD app that only displays numbers. Every screen is backed by logic:

- **Priority scoring** — each order gets a 0–100+ score from SLA proximity, account
  tier, order value, order age, and operational flags in the notes (e.g. "fleet down").
  Labeled URGENT / HIGH / NORMAL / LOW.
- **Allocation under scarcity** — when two orders compete for the same SKU and stock
  is short, the engine allocates to the higher-priority order first (fully, if
  possible), then partially allocates or backorders the rest — exactly the
  10-vs-7-units scenario from the brief. Every allocation decision is logged in
  plain language ("ORD-8841 (URGENT) — partial allocation for WH-1002: got 8/10,
  short 2. Backordered pending replenishment.").
- **Reorder recommendations** — recommended reorder quantity is computed from
  average daily usage, supplier lead time, and a safety-stock buffer, not just a
  static threshold.
- **Pick-path optimization** — pick lists are sorted by zone/aisle/bin to reduce
  walking distance, instead of listing items in order-entry order.
- **Exception → Decision → Resolution** — reporting a damaged or missing item
  immediately deducts stock, checks whether it pushes the SKU below its reorder
  point, flags every open order that references that SKU, and proposes next steps.
- **Bottleneck detection** — analytics highlight the slowest stage in the pipeline
  (mock stage-duration data) and suggest an operational fix.

## Tech notes

- Plain HTML + React 18 (via CDN, in-browser Babel) + Chart.js — chosen so the
  whole thing is a single portable file for judging, with no build tooling.
- All data is mock/in-memory (per the problem statement) and resets on page reload.
- Visual design: an industrial "warehouse signage" palette (steel dark background,
  safety-orange accent, hazard-stripe dividers, bin-location tags) with Barlow
  Condensed for display type and Inter/JetBrains Mono for body and data.
