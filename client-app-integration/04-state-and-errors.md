# State and errors

## Suggested frontend state slices

| Slice | Contents |
| --- | --- |
| `session` | current client, auth bootstrap status |
| `catalog` | filters, search string, current page, products |
| `favorites` | favorite ids and paged favorite list |
| `cart` | grouped cart items, totals, pending mutations |
| `deliveryAddresses` | saved addresses and selected default |
| `orders` | order list, detail, action loading states |
| `disputes` | dispute list, detail, initial chat history, realtime messages |
| `notifications` | inbox pages, unread state, detail cache |

## Realtime rule

For disputes and notifications, the preferred model is:

1. initial REST read;
2. websocket subscription;
3. local cache updates.

Do not rely on periodic polling as the main synchronization strategy.

## Error handling pattern

Backend business errors use a stable JSON shape:

```json
{
  "statusCode": 422,
  "statusKey": 422001,
  "message": "",
  "error": "",
  "field": "delivery_address_id",
  "constraint": "isUuid"
}
```

## UI guidance

- Use `statusCode` for broad handling and `statusKey` for product-specific behavior.
- Show inline field errors when `field` is present.
- Treat `401` as a session/bootstrap issue.
- Treat `403` as a permissions/state restriction.
- Treat `404` as stale deep link or deleted entity.
- Treat `409` as business conflict.
- Treat `422` as validation or state-precondition failure.

## Mutation strategy

- Optimistic updates are safe for local UI toggles like favorites and read-state when rollback is easy.
- Prefer cache patching over broad invalidation after realtime events.
- For checkout and dispute creation, prefer pessimistic UX until the backend confirms the final payload.
