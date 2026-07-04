---
name: retail-cart-mirror
description: Mirror retail buy lists into shopping carts for legitimate bulk purchase preparation and line-by-line cart verification. Use when the user provides a spreadsheet, CSV, pasted table, or buy list and asks to add items to cart, verify colours/sizes, handle user-approved substitutions, record product links/statuses, or coordinate a hybrid human-agent retail cart workflow. Requires normal user-visible browser interaction such as Playwright MCP when automating; the human owns login, human verification, checkout, payment, and final purchase. Do not use for checkout/payment submission, credential handling, unauthorized scraping, rate-limit evasion, anti-bot/CAPTCHA bypass, credential stuffing, sneaker/ticket botting, or attacks.
---

# retail-cart-mirror

## One Cut

Mirror a structured retail buy list into a live cart, then prove each cart line matches the requested item, colour, size, quantity, or an explicitly approved substitution.

The skill is not a generic shopping assistant. It exists to prevent ordering drift.

## Stance

Act like a cart auditor with browser hands. The human owns authority; the agent owns repetition and evidence.

```text
source row -> selected variant -> cart line -> ledger verdict
```

No cart line evidence, no success claim.

## Load when needed

- Read `references/status-codes.md` before building or updating the Cart Ledger.
- Read `references/safety-boundaries.md` when the task touches login, human verification, checkout, rate limits, site controls, or unclear legitimacy.
- Use `templates/cart-ledger.md` for the running ledger.
- Use `templates/final-verdict.md` before final handoff.

## Required input packet

Before adding anything, establish:

```text
Source list:
- file/table/list location
- columns or fields for item, colour, size, quantity, notes

Cart policy:
- clear existing cart? yes/no
- one consolidated session/cart? yes/no
- stop before checkout/payment? always yes

Matching policy:
- exact item number required? yes/no
- exact colour required? yes/no
- exact size required? yes/no
- allowed substitutions, if any

Human boundary:
- user handles login/human verification/checkout/payment
- agent stops or waits when blocked

Writeback:
- no writeback / spreadsheet / notes / status table
```

If substitution rules are missing, ask before substituting. Never infer a larger or smaller size silently.

## Process map

```text
BUY LIST
   |
   v
NORMALIZE ROWS
item, colour, size, qty, substitution policy
   |
   v
POLICY LOCK
cart baseline, human boundary, no checkout
   |
   v
ROW LOOP
search -> product -> colour -> size -> add / block / user-add marker
   |
   v
RUNNING LEDGER
status, URL/SKU/cart count evidence, notes
   |
   v
CART MIRROR
extract cart product, colour, size, SKU, qty
   |
   v
RECONCILE
source rows <-> cart lines
   |
   v
FINAL VERDICT
ready for human review / blocked / needs fix
```

## Procedure

### 1. Normalize the buy list

Create one row per requested variant. Preserve original row IDs from the source when available.

Track:

```text
row_id, item_number, requested_colour, requested_size, quantity, substitution_allowed, notes
```

Flag before browsing:

- blank colours,
- size ranges,
- duplicate requested variants,
- ambiguous notes,
- rows marked already ordered / out of stock / skipped,
- non-standard size aliases such as `5T`, `5Y`, `XL`, `4/5`.

### 2. Lock policy and baseline cart

Confirm whether to clear or preserve the cart. If the user needs one purchase, keep one browser session/cart.

Record baseline:

```text
cart_url:
initial_cart_count:
cart_cleared_or_preserved:
```

### 3. Resolve each source row

For each row:

1. Search or navigate by item number or user-provided URL.
2. Confirm the product page is relevant to the requested item.
3. Select exact colour. If absent, mark `COLOUR_MISMATCH`.
4. Select exact size or approved substitution. If unavailable, mark `OOS` or `SIZE_MISMATCH`.
5. Add only unambiguous variants to cart.
6. Capture evidence: URL, visible product name, selected colour, selected size, SKU if visible, add confirmation/cart count.

Do not treat a search result as proof. Proof comes from selected product state or cart line extraction.

### 4. Handle human verification and manual adds

If the site requires login, CAPTCHA, press-and-hold, payment, address entry, or other human-owned steps, stop and give a handoff packet:

```text
Blocked row:
URL:
Last completed row:
What the user should do:
How to resume:
```

If the user says they manually added a row, mark `USER_ADDED_PENDING_VERIFY`. Upgrade it only after the cart mirror audit proves matching product/colour/size/qty.

### 5. Maintain the Cart Ledger

Use the status vocabulary in `references/status-codes.md`.

Every row needs one status and one evidence note. Missing evidence means `MANUAL_REVIEW`, not `ADDED`.

### 6. Mirror the cart

Before final handoff, inspect the live cart and extract visible cart lines:

```text
cart_line_id, product_name, colour, size, sku, quantity, price_if_needed
```

Then reconcile source rows to cart lines.

Flag:

- expected row missing from cart,
- extra cart line not mapped to a source row,
- wrong colour,
- wrong size,
- duplicate quantity issue,
- substitution not explicitly approved,
- user-added item not verified.

### 7. Final verdict

Use `templates/final-verdict.md`.

The final answer must separate:

- exact matches,
- accepted substitutions,
- user-added verified rows,
- not found/OOS/mismatches,
- unresolved manual review items.

End with whether the cart is ready for **human review**, not purchase.

## Redlines

- Never checkout, purchase, submit payment, or place an order.
- Never enter, edit, reveal, or store credentials, addresses, payment cards, gift cards, or loyalty data.
- Never bypass CAPTCHA, human verification, queues, rate limits, bot detection, or platform controls.
- Never use rotating proxies, stealth tooling, hidden browser behavior, credential stuffing, or attack-like automation.
- Never claim an item is correct without selected-variant or cart-line evidence.
- Never silently substitute colour, size, product, or quantity.
- Never continue when legality, authorization, or site-control boundaries are unclear; ask or stop.
- Never write back to a spreadsheet/source file unless the user asked for writeback and the target columns are clear.

## Good final shape

```text
Final Cart Verdict
- Cart count: 13
- Source rows processed: 20
- Exact matches: 12
- Accepted substitutions: 1
- User-added verified: 2
- Not found/OOS/mismatch: 7
- Unresolved manual review: 0
- Checkout/payment touched: no
- Ready for human review: yes
```
