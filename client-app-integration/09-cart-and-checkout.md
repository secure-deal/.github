# Cart and checkout

## Cart routes

| Method | Route | Purpose |
| --- | --- | --- |
| `GET` | `/api/client/cart` | Current cart grouped by merchant. |
| `POST` | `/api/client/cart` | Add or increment cart item. |
| `POST` | `/api/client/cart/upsert` | Replace quantity for a product. |
| `PUT` | `/api/client/cart/:id` | Update item quantity. |
| `DELETE` | `/api/client/cart/:id` | Remove one item. |
| `DELETE` | `/api/client/cart` | Clear cart manually. |

## Checkout routes

| Need | Route | Body |
| --- | --- | --- |
| Full current cart | `POST /api/client/orders/batch_create` | `{ "delivery_address_id": "uuid" }` |
| One merchant slice | `POST /api/client/orders` | `{ "delivery_address_id": "uuid", "merchant_slug": "techno-plus" }` |

## Checkout rules

1. Checkout requires a saved delivery address.
2. Checkout is cart-based; UI does not send line items directly.
3. Backend groups cart items by merchant and creates one order per merchant group.
4. Delivery fee is currently `15` per created order.
5. Stock is validated at checkout time.
6. Successful checkout returns `orders[]`.
7. **Cart is preserved after checkout. Do not assume the backend will empty it.**

## Delivery address requirement

Create/update body:

```json
{
  "city": "Dushanbe",
  "location": "Rudaki ave, 12",
  "longitude": 68.7739,
  "latitude": 38.5598,
  "is_default": true
}
```

## Success response

```json
{
  "orders": [
    {
      "id": "uuid",
      "order_id": "1000000000001",
      "merchant_id": "uuid",
      "merchant_store_name": "TechnoPlus",
      "items_total": 200,
      "delivery_fee": 15,
      "total_amount": 215,
      "status": "paid",
      "payment_status": "paid"
    }
  ],
  "total_amount": 215
}
```

## Frontend implications

- Show checkout success as a list of created orders, not as one implicit order.
- Do not use `/api/client/cart/checkout`; active checkout lives under `/api/client/orders`.
- After checkout, refresh cart deliberately if the UI wants to show current remaining cart contents.
- Handle stock/address errors by refreshing the relevant screen state.
