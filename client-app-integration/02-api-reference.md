# Client API reference

All routes are relative to the backend origin and use `/api/client/*`.

Authenticated browser requests must use cookies (`credentials: "include"`).

## Checkout and cart

| Method | Route | Purpose |
| --- | --- | --- |
| `GET` | `/api/client/cart` | Current cart. |
| `POST` | `/api/client/cart` | Add/increment item. |
| `POST` | `/api/client/cart/upsert` | Replace quantity by product id. |
| `PUT` | `/api/client/cart/:id` | Update quantity by cart-item id. |
| `DELETE` | `/api/client/cart/:id` | Remove cart item. |
| `DELETE` | `/api/client/cart` | Clear cart manually. |
| `POST` | `/api/client/orders/batch_create` | Checkout full current cart. |
| `POST` | `/api/client/orders` | Checkout selected merchant cart slice. |

Checkout request:

```json
{
  "delivery_address_id": "uuid"
}
```

Single-merchant checkout:

```json
{
  "delivery_address_id": "uuid",
  "merchant_slug": "techno-plus"
}
```

Important:

- both checkout routes still read from the current cart;
- one checkout can create several orders;
- cart items remain in the cart after successful order creation.

## Delivery addresses

| Method | Route |
| --- | --- |
| `GET` | `/api/client/delivery-addresses` |
| `GET` | `/api/client/delivery-addresses/cities` |
| `POST` | `/api/client/delivery-addresses` |
| `GET` | `/api/client/delivery-addresses/:id` |
| `PUT` | `/api/client/delivery-addresses/:id` |
| `DELETE` | `/api/client/delivery-addresses/:id` |

## Orders

| Method | Route | Purpose |
| --- | --- | --- |
| `GET` | `/api/client/orders` | Current-client order list. |
| `GET` | `/api/client/orders/:id` | Order detail by UUID. |
| `PUT` | `/api/client/orders/:id/received` | Start inspection for delivered order. |
| `POST` | `/api/client/orders/:id/complete-delivery` | Immediate completion shortcut for delivered order. |
| `POST` | `/api/client/orders/:id/confirm-inspection` | Complete inspection flow. |

Order detail fields the client app should consume:

- `id`, `order_id`
- `merchant_id`, `merchant_slug`, `merchant_store_name`
- `client_id`, `client_phone`, `client_name`
- `delivery_address_id`
- `delivery_address_location`
- `status`, `payment_status`, `status_rendering`
- `items`
- `paid_at`, `delivered_at`, `inspection_ends_at`, `completed_at`, `cancelled_at`

## Disputes

| Method | Route |
| --- | --- |
| `POST` | `/api/client/disputes` |
| `GET` | `/api/client/disputes` |
| `GET` | `/api/client/disputes/:id` |
| `GET` | `/api/client/disputes/:id/chat/messages` |
| `POST` | `/api/client/disputes/:id/chat/messages` |

Use dispute UUIDs for detail and chat routes.

`POST /api/client/disputes` is valid only when the target order status is `delivered`, `inspection`, or `completed`.

Legacy order-scoped dispute routes still exist, but new UI should use the dedicated dispute API.

## Notifications

| Method | Route |
| --- | --- |
| `GET` | `/api/client/notifications` |
| `GET` | `/api/client/notifications/:id` |
| `POST` | `/api/client/notifications/:id/read` |
| `POST` | `/api/client/notifications/read-all` |
| `PATCH` | `/api/client/notifications/read-state` |

Realtime updates arrive through `/notifications` with `notification.created`.
