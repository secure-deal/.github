# Client app integration

This folder is the frontend-oriented guide for the buyer application. It keeps practical docs that help a frontend team wire screens, API calls, state, and realtime behavior without digging through the whole backend.

## Guide map

| File | Purpose |
| --- | --- |
| `01-screens-and-routes.md` | Reference route map based on the current buyer demo app. |
| `02-api-reference.md` | Buyer API surface summary. |
| `03-flows.md` | Core end-to-end user flows. |
| `04-state-and-errors.md` | Suggested frontend state boundaries and error handling. |
| `05-implementation-checklist.md` | Practical integration checklist. |
| `06-auth.md` | Cookie auth contract and session behavior. |
| `07-catalog-search-filtering.md` | Catalog, search, categories, and filter payloads. |
| `08-favorites-and-reviews.md` | Favorite products and review flows. |
| `09-cart-and-checkout.md` | Cart grouping, addresses, and checkout behavior. |
| `10-orders-lifecycle.md` | Buyer order states and order detail contract. |
| `11-disputes.md` | Buyer disputes and dispute chat contract. |
| `12-notifications.md` | Buyer inbox and realtime notifications. |
| `13-validation-errors.md` | Error payload shape and common validation cases. |

## Current backend truths

1. Checkout is cart-based and requires `delivery_address_id`.
2. Single-merchant checkout uses `POST /api/client/orders` with `merchant_slug`.
3. Batch checkout uses `POST /api/client/orders/batch_create`.
4. Cart is **not** auto-cleared after order creation.
5. Client order detail includes `delivery_address_id` and `delivery_address_location`.
6. Active dispute routes use dispute UUIDs.
7. Realtime UX should be modeled as **initial REST fetch + websocket updates**.
