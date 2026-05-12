# Notifications

Notifications are durable inbox rows for the current client.

## API

| Method | Route |
| --- | --- |
| `GET` | `/api/client/notifications` |
| `GET` | `/api/client/notifications/:id` |
| `POST` | `/api/client/notifications/:id/read` |
| `POST` | `/api/client/notifications/read-all` |
| `PATCH` | `/api/client/notifications/read-state` |

## Payload

The client app should support:

- `id`
- `category`
- `eventType`
- `kind`
- `title`
- `message`
- `body`
- `priority`
- `isRead`
- `readAt`
- `createdAt`
- `actions`

## Realtime contract

The recommended client pattern is:

1. initial inbox fetch via REST;
2. subscribe to `/notifications`;
3. prepend new `notification.created` payloads into cache;
4. update read-state caches locally after `read`, `read-all`, or bulk updates.

Repeated REST polling is not the normal refresh mechanism.

Socket settings:

| Setting | Value |
| --- | --- |
| Namespace | `/notifications` |
| Event | `notification.created` |
| Auth | existing HttpOnly cookies with `withCredentials: true` |

Example:

```ts
import { io } from "socket.io-client";

const socket = io("/notifications", {
  path: "/socket.io",
  withCredentials: true,
  transports: ["websocket", "polling"],
});

socket.on("notification.created", (payload) => {
  // payload matches NotificationResponse
});
```

## Event families

| Event type | Meaning |
| --- | --- |
| `order.created` | New order created |
| `order.status.changed` | Order state changed |
| `order.inspection.started` | Inspection window opened |
| `payment.confirmed` | Payment succeeded |
| `payment.pending` | Payment still processing |
| `payment.failed` | Payment failed |
| `dispute.opened` | Dispute opened |
| `dispute.assigned` | Operator assigned |
| `dispute.status.changed` | Dispute status/resolution changed |
| `dispute.message.created` | New dispute chat message |
