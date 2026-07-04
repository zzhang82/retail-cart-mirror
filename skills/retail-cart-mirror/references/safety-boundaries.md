# Safety boundaries

This skill supports legitimate, user-authorized retail purchasing preparation. It does not grant the agent authority to bypass platform controls or make purchases.

## Allowed

- Read a user-provided buy list.
- Use normal, user-visible browsing to find products.
- Select product variants when item, colour, size, and quantity are unambiguous.
- Add available items to a cart for later human review.
- Record product URLs, visible SKUs, cart-line colours, sizes, and statuses.
- Pause while the user handles login, human verification, checkout, payment, or account-specific steps.
- Verify items the user manually added by inspecting visible cart lines.

## Not allowed

- Placing orders or submitting payment.
- Entering, changing, exposing, or storing credentials, addresses, payment cards, gift cards, or loyalty data.
- Bypassing CAPTCHA, press-and-hold checks, bot detection, rate limits, queues, or access controls.
- Using stealth automation, residential proxies, rotating identities, hidden browsers, or solver services.
- Automating limited-drop, ticket, sneaker, queue, or unfair inventory capture workflows.
- Scraping product catalogs unrelated to the user's provided list.
- Continuing when the user has not established legitimate authority to purchase.

## Hybrid handoff rule

When blocked by a human-owned step, stop with:

```text
Blocked at:
Current source row:
Last completed source row:
What the human should do:
What evidence to check after resume:
```

After the user resumes or manually adds an item, verify the cart line before upgrading the row status.
