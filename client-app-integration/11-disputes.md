# Disputes

## Active routes

| Method | Route | Purpose |
| --- | --- | --- |
| `POST` | `/api/client/disputes` | Create dispute. |
| `GET` | `/api/client/disputes` | List current client disputes. |
| `GET` | `/api/client/disputes/:id` | Detail by dispute UUID. |
| `GET` | `/api/client/disputes/:id/chat/messages` | Initial chat history. |
| `POST` | `/api/client/disputes/:id/chat/messages` | Send chat message. |

## Identifier rule

Active client flows use the dispute UUID. Treat `id` as the canonical dispute identifier for detail pages and chat.

## When the client can open a dispute

The backend allows `POST /api/client/disputes` only when the order is currently in one of these statuses:

| Order status | Meaning for the client app |
| --- | --- |
| `delivered` | The order was marked as delivered and the buyer can immediately escalate if there is a problem. |
| `inspection` | The inspection window is open; dispute creation is allowed during this period. |
| `completed` | A dispute can still be opened even after order completion. |

For all other order statuses, opening a dispute should be treated as unavailable in the UI.

## Detail response

Render:

- `id`
- `order_id`
- `client_name`, `client_phone`
- `merchant_title`
- `operator_name`
- `cause`, `description`
- `media`
- `operator_resolution_details`
- `status`
- `opened_at`, `resolved_at`
- `order_price`

## Chat contract

Rules:

1. Load the initial message history via REST.
2. After that, append incoming websocket messages instead of polling the history endpoint.
3. Message or attachment must be present.
4. Deduplicate by message `id`.
5. Resolved disputes are read-only.

## Realtime

| Setting | Value |
| --- | --- |
| Namespace | `/disputes-chat` |
| Join event | `dispute:join` |
| Join payload | `{ "id": "<dispute-uuid>" }` or `{ "dispute_id": "<dispute-uuid>" }` |
| Events | `dispute.chat.message.created`, `dispute.chat.message.system` |

Connection pattern:

```ts
import { io } from "socket.io-client";

const socket = io("/disputes-chat", {
  path: "/socket.io",
  withCredentials: true,
  transports: ["websocket", "polling"],
});
```

Join once initial REST data is ready, then re-join after reconnect if the client reconnects.
