# Implementation checklist

## Session and transport

- [ ] All authenticated requests send cookies with `credentials: "include"`.
- [ ] App bootstraps client session from `/api/client/auth/me`.
- [ ] Logout clears client-side state after `POST /api/client/auth/logout`.

## Catalog

- [ ] Category tree and category attributes are loaded from the client category APIs.
- [ ] Catalog listing uses `POST /api/client/catalog`.
- [ ] Product detail uses product slug.
- [ ] Similar products and product reviews are wired.

## Cart and checkout

- [ ] Cart mutations use the client cart API only.
- [ ] Delivery address is required before checkout.
- [ ] Checkout flow handles multi-order success response.
- [ ] UI does not assume the cart is empty after checkout.

## Orders

- [ ] Order detail uses UUID route params.
- [ ] UI renders `delivery_address_id` and `delivery_address_location` where needed.
- [ ] Order actions are driven by backend status / `status_rendering`.

## Disputes

- [ ] New disputes use `POST /api/client/disputes`.
- [ ] Detail/chat pages use dispute UUIDs.
- [ ] Initial chat history comes from REST.
- [ ] Ongoing chat updates come from WS room events.

## Notifications

- [ ] Initial inbox uses REST.
- [ ] Realtime uses `/notifications`.
- [ ] Read-state updates patch local cache and unread counters.

## Quality

- [ ] API errors are surfaced with field-level feedback where possible.
- [ ] Empty/loading/error states exist for list and detail pages.
- [ ] Deep links handle `404`, expired session, and forbidden transitions cleanly.
