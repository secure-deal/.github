# Secure Deal docs

This folder is the trimmed, current documentation set. It keeps only the docs that still describe active behavior in the codebase.

## Structure

| Path                                             | Purpose                                                            |
| ------------------------------------------------ | ------------------------------------------------------------------ |
| `specs/orders.md`                                | Current order and checkout contract.                               |
| `specs/employee-warehouse-ops.md`                | Employee warehouse/PVZ API: lookup, queue, transitions, QR handoff. |
| `specs/disputes.md`                              | Current dispute lifecycle, chat, and operator resolution contract. |
| `specs/notifications.md`                         | Current inbox, eventing, and realtime notification contract.       |
| `specs/product-attributes/*`                     | Active product-attribute contracts.                                |
| `client-app-integration/README.md`               | Buyer integration entrypoint.                                      |
| `client-app-integration/02-api-reference.md`     | Buyer-facing API summary.                                          |
| `client-app-integration/09-cart-and-checkout.md` | Cart, address, and checkout behavior.                              |
| `client-app-integration/10-orders-lifecycle.md`  | Buyer order states and actions.                                    |
| `client-app-integration/11-disputes.md`          | Buyer dispute detail/chat contract.                                |
| `client-app-integration/12-notifications.md`     | Buyer inbox + realtime notification contract.                      |

## Cleanup rules

- Deleted sections were removed intentionally because they were archival, duplicated, or stale.
- Code in `app/` remains the final source of truth.
- This docs set focuses on active runtime contracts, not historical plans.

## Recent behavior reflected here

- Cart checkout no longer clears cart items automatically after order creation.
- Client order detail exposes `delivery_address_id` and `delivery_address_location`.
- Disputes use UUID-based identifiers in active REST/UI flows.
- Realtime UX is modeled as **initial REST fetch + websocket updates**, not repeated polling loops.
