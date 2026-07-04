# retail-cart-mirror

`retail-cart-mirror` is a public agent skill for **legitimate retail bulk purchasing preparation**: take a buy list, add only matching available variants to a shopping cart, and prove the final cart mirrors the source list before a human reviews and purchases.

The skill is designed for workflows such as reseller replenishment, family/business bulk ordering, team apparel ordering, or procurement assistants where the user is allowed to buy the products and wants help with repetitive variant matching.

It is **not** a botting, scraping, checkout, credential, or anti-abuse bypass skill.

## What it does

```text
BUY LIST
  -> normalize item / colour / size / qty / substitution rules
  -> search/select only unambiguous variants
  -> add available items to cart
  -> record each row's status and evidence
  -> extract cart lines
  -> reconcile cart lines back to the source list
  -> hand the cart to the human for final review and purchase
```

Core output: a **Cart Ledger** that records every source row as `ADDED`, `USER_ADDED_PENDING_VERIFY`, `USER_ADDED_VERIFIED`, `ACCEPTED_SUBSTITUTE`, `NOT_FOUND`, `OOS`, `COLOUR_MISMATCH`, `SIZE_MISMATCH`, `AMBIGUOUS_BLOCKED`, `HUMAN_VERIFY_BLOCKED`, or `MANUAL_REVIEW`.

## Legitimate use vs bot abuse

This skill is for a hybrid human/agent workflow where:

- The user has a legitimate reason and right to purchase the goods.
- The user controls the account, login, shipping, payment, final review, and purchase decision.
- The agent helps with repetitive list-to-cart matching and evidence capture.
- Human verification, CAPTCHA, login, payment, and checkout remain human-owned steps.
- Site limits, robots, terms, and normal browser behavior are respected.

This skill must **not** be used for:

- account takeover, credential stuffing, checkout automation, or payment submission;
- bypassing CAPTCHA, human verification, rate limits, queues, or anti-bot controls;
- sneaker/ticket/limited-drop botting or unfair inventory capture;
- unauthorized scraping or extraction beyond what is needed to complete the user's visible cart workflow;
- hiding automation identity, rotating proxies, or evading platform controls.

## Hybrid capture model

The safest and most useful pattern is hybrid:

```text
Human
  owns account / verification / checkout / final purchase

Agent
  owns repetitive variant search, status ledger, cart extraction, and reconciliation

Shared handoff
  if blocked, user clears the gate or manually adds an item;
  agent later verifies the cart line before claiming success
```

This matters because the goal is not to “beat the site.” The goal is to prevent human ordering mistakes: wrong colour, wrong size, duplicate row, forgotten not-found item, or silent substitution.

## Prerequisites

- An agent runtime that supports skills, such as Codex/OpenCode-style skill folders.
- Browser automation available through normal user-visible browsing. **Playwright MCP is recommended** because it can navigate, click, inspect cart DOM lines, and preserve one browser session.
- A source buy list, usually XLSX, CSV, pasted table, or a manually transcribed list.
- User-provided matching policy, especially allowed substitutions such as `5T -> 5` or `4T -> 5`.

If Playwright MCP is unavailable, the skill can still be used as a manual reconciliation checklist, but the agent should not claim cart verification without cart-line evidence.

## Installation

Clone this repo and copy or symlink the skill folder into your local skill directory:

```bash
git clone https://github.com/<owner>/retail-cart-mirror.git
cp -R retail-cart-mirror/skills/retail-cart-mirror ~/.config/opencode/skills/
```

Adjust the destination for your agent runtime if it uses a different skill directory. The repo is intentionally runtime-light: the reusable artifact is the `skills/retail-cart-mirror/` folder, so other skill-capable agents can copy that subtree into their own skill/plugin location.

## Example prompt

```text
Use retail-cart-mirror. I have a spreadsheet with item number, colour, and size.
Clear the cart first, add available exact matches, allow 5T -> 5, stop before checkout,
and give me a final cart ledger with not-found/OOS/mismatch rows.
```

## Repository layout

```text
skills/retail-cart-mirror/
  SKILL.md
  agents/openai.yaml
  references/status-codes.md
  references/safety-boundaries.md
  templates/cart-ledger.md
  templates/final-verdict.md
```

There is intentionally no retailer-specific reference file. Site-specific notes belong in a private project repo or local memory, not in this public skill.
