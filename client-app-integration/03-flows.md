# Core flows

## 1. Auth bootstrap

1. Submit `POST /api/client/auth/login` with `bank_id`, `phone`, and `name`.
2. Backend sets auth cookies.
3. Read `GET /api/client/auth/me` to hydrate the session.
4. Keep subsequent requests cookie-based with `credentials: "include"`.

## 2. Browse and search

1. Load category tree from `/api/client/categories/tree`.
2. Load category attributes for selected category when filters are needed.
3. Request product list via `POST /api/client/catalog`.
4. When the buyer searches, also use `/api/client/catalog/search/suggestions` and recent-search endpoints if the user is authenticated.

## 3. Favorite from listing or product page

1. Call `POST /api/client/favorites/products/:slug`.
2. Update the local product card/detail state optimistically if desired.
3. Refresh or patch the favorites collection cache.

## 4. Cart to checkout

1. Add/update cart items with `/api/client/cart`.
2. Ensure the buyer has at least one delivery address.
3. Submit checkout through `/api/client/orders/batch_create` or `/api/client/orders`.
4. Render the returned `orders[]`.
5. Do not assume cart auto-clears after success.

## 5. Order follow-up

1. Load `/api/client/orders` for list.
2. Load `/api/client/orders/:id` for detail.
3. Start inspection with `PUT /api/client/orders/:id/received` when applicable.
4. Complete with `/complete-delivery` or `/confirm-inspection` when the action becomes available.

## 6. Dispute flow

1. Open dispute with `POST /api/client/disputes` only when the order status is `delivered`, `inspection`, or `completed`.
2. Show dispute detail by dispute UUID.
3. Load initial chat history via REST.
4. Join WS room and append new messages locally.
5. Stop input when dispute is resolved.

## 7. Notifications flow

1. Load initial inbox via REST.
2. Subscribe to `/notifications`.
3. Prepend `notification.created` payloads to local cache.
4. Patch read state locally after read / read-all mutations.
