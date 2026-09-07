# Mart Section — Design Spec

Date: 2026-09-07

## Goal

Add a bulk-goods "Mart" section to the existing FoodGuard app (`home/index.html`), where users browse bulk-only grocery products classified into nutrition groups (High Protein, High Carbs, High Fat, Balanced), filter/search them, and add to a cart (no real payment). The existing pantry/expiry/nutrition tracker is unchanged; Mart is a new view alongside it.

## Scope

- New "Mart" view added to `home/index.html`'s existing single-page view system (`Dashboard`, `Inventory`, `Nutrition`, `Alerts`, `Insights` → + `Mart`).
- New sidebar entry and mobile-nav entry for Mart.
- Cart icon + badge count in the top bar, opening a cart modal.
- No new files, no build tooling, no framework. Stays plain HTML/CSS/JS + `localStorage`, matching the file's current architecture.
- `index.html` (marketing landing page) is untouched — this work is scoped to `home/index.html` only.

## Out of scope

- Real payment/checkout processing (checkout button shows a toast confirmation only).
- Backend/API — product data is a hardcoded in-file array, same pattern as the existing `seed` food array.
- User accounts, order history, persistence beyond the current browser's `localStorage`.

## Data model

```js
// group: 'high-protein' | 'high-carb' | 'high-fat' | 'balanced'
{ id, name, group, packSize, price, caloriesPer100g, proteinPer100g, carbsPer100g, fatPer100g, icon }
```

All `packSize` values are bulk sizes only (5kg/10kg/25kg bags or tubs/tins) — no single-unit or retail-size products. Prices in ₹ (INR), flat numbers, no decimals, matching the app's existing informal/demo tone (calorie/macro values in the seed data are similarly simplified).

### Product catalog (19 items)

**High Protein**
1. Whey Protein Powder — 5kg tub — ₹8500
2. Chana (Chickpeas) — 25kg bag — ₹2800
3. Soybean Chunks — 10kg bag — ₹1600
4. Paneer Block (Bulk) — 5kg — ₹2200
5. Raw Peanuts — 10kg bag — ₹1400
6. Moong Dal — 25kg bag — ₹3200

**High Carbs**
7. Basmati Rice — 25kg bag — ₹2600
8. Wheat Flour (Atta) — 25kg bag — ₹1100
9. Rolled Oats — 10kg bag — ₹1300
10. Potatoes — 25kg bag — ₹700
11. Brown Rice — 25kg bag — ₹2400

**High Fat**
12. Ghee — 5kg tin — ₹4200
13. Almonds — 5kg bag — ₹4800
14. Peanut Butter — 5kg tub — ₹1900
15. Coconut Oil — 5L tin — ₹1500
16. Cashews — 5kg bag — ₹5200

**Balanced**
17. Mixed Dal Combo — 10kg bag — ₹1500
18. Multigrain Atta — 25kg bag — ₹1350
19. Mixed Vegetables (Bulk Pack) — 10kg crate — ₹900

Macro values per 100g are approximate real-world values (e.g. whey ~80g protein/100g, rice ~28g carbs/100g cooked-basis simplified to dry-basis for this demo) — reasonable, not lab-precise; this is a prototype, matching the existing app's disclaimer tone ("demonstration values for this college-project prototype").

## UI

**Sidebar / mobile nav**: new "🛒 Mart" entry, same styling as existing nav buttons.

**Mart view**:
- Header: eyebrow "Bulk marketplace", headline "Stock up in bulk.", sub-copy.
- Filter chip row: `All`, `High Protein`, `High Carbs`, `High Fat`, `Balanced` — pill buttons, active state mirrors existing `.nav button.active` treatment.
- Search bar reusing `.searchbar`/`.input` pattern, filters by name.
- Product grid (`.grid`, responsive columns) of `.card` elements, one per product:
  - icon, name, group badge (color-coded: protein=safe/green, carbs=warn/amber, fat=danger/red-ish or a new neutral tone, balanced=a 4th neutral badge color added to CSS)
  - pack size + price
  - macros per 100g (small text row: cal/protein/carb/fat)
  - qty stepper (−/number/+) and "Add to cart" button

**Cart**:
- Icon button in top bar (next to "+ Add food"), badge shows total item count.
- Click opens a modal (reusing `.modal-wrap`/`.modal`) listing cart lines: icon, name, pack size, qty (editable), line price, remove button.
- Subtotal at the bottom.
- "Checkout" button: clears cart, shows toast "Order placed (demo) — thank you!", closes modal. No real payment.
- Cart persisted in `localStorage` key `foodguard-cart` (array of `{productId, qty}`), same `save()`-style pattern as existing `foods`/`consumed`.

## Interaction / data flow

- Filter chips and search box both narrow the same product list (AND logic) — same UX as existing Inventory search+filter.
- "Add to cart" on a product: if already in cart, increments qty by the stepper amount; else adds new line at qty from stepper (default 1).
- Cart badge and modal content re-render via the same central `render()` dispatcher the app already uses — add `renderMart()` and `renderCart()` calls into it, plus call `renderMart()`/badge update on qty/cart changes without needing a full view switch.
- Removing the last unit of a line via qty stepper removes the cart line entirely.

## Error handling / edge cases

- Empty cart: modal shows the existing `.empty` state pattern ("Your cart is empty.").
- Empty filter/search result: product grid shows `.empty` state ("No bulk products match.").
- Checkout with empty cart: button disabled (or no-op) rather than showing a confirmation toast for nothing.
- No network/backend calls introduced — nothing to fail beyond normal JS logic, kept dependency-free like the rest of the file.

## Testing / verification

Since this is a static HTML/JS app with no test harness, verification is manual via `run` skill / browser:
- Load `home/index.html`, navigate to Mart tab, confirm product grid renders all 19 products, correct group badges.
- Filter by each of the 4 groups, confirm correct subset shown.
- Search by partial product name, confirm filter narrows correctly combined with active chip.
- Add multiple products to cart (including qty > 1), open cart modal, confirm lines/subtotal correct.
- Remove a line, edit qty to 0, confirm line disappears and totals update.
- Checkout, confirm cart clears, toast shows, badge resets to 0.
- Reload page, confirm cart persists (localStorage) if items were left in cart before reload — confirm cleared cart also persists as empty after checkout.
- Confirm existing views (Dashboard/Inventory/Nutrition/Alerts/Insights) unaffected — no regressions from the new nav entry or added CSS.
