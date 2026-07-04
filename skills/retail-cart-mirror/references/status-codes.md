# Status codes

Use one status per source row.

| Status | Meaning | Required evidence |
| --- | --- | --- |
| `ADDED` | Agent added an exact requested variant. | Product/cart line with matching colour, size, and quantity. |
| `USER_ADDED_VERIFIED` | User manually added the row and the cart audit verified it. | Cart line with matching colour, size, and quantity. |
| `USER_ADDED_PENDING_VERIFY` | User says they added it, but the cart audit has not verified it yet. | User statement plus pending cart check. |
| `ACCEPTED_SUBSTITUTE` | Cart line differs from request but matches explicit substitution policy or user approval. | Cart line plus substitution note. |
| `NOT_FOUND` | Item number/search could not locate a plausible product. | Search URL/result or product lookup note. |
| `OOS` | Product/colour exists but requested size or variant is unavailable/out of stock. | Product URL and unavailable size/variant note. |
| `COLOUR_MISMATCH` | Product exists but requested colour is absent. | Product URL and available colour note. |
| `SIZE_MISMATCH` | Requested size is absent and no approved substitution applies. | Product URL and available size note. |
| `AMBIGUOUS_BLOCKED` | More than one plausible match or source row is unclear. | Ambiguity note and what confirmation is needed. |
| `HUMAN_VERIFY_BLOCKED` | Site requires a human-owned gate before continuing. | URL, blocked row, and resume instruction. |
| `MANUAL_REVIEW` | Evidence is incomplete or mapping is uncertain. | Reason the row cannot be safely classified. |

Downgrade rules:

- `ADDED` becomes `MANUAL_REVIEW` if final cart evidence is missing.
- `USER_ADDED_PENDING_VERIFY` becomes `USER_ADDED_VERIFIED` only after cart-line reconciliation.
- Any unapproved substitution is `SIZE_MISMATCH`, `COLOUR_MISMATCH`, or `MANUAL_REVIEW`, not `ACCEPTED_SUBSTITUTE`.
