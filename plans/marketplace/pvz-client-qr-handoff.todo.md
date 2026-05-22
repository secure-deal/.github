# PVZ client QR handoff — progress

> Plan: [pvz-client-qr-handoff.md](./pvz-client-qr-handoff.md)

## Phase 4.0 — MVP

### Backend: token & client API

- [x] `PVZ_HANDOFF_SECRET` (or document reuse of `JWT_SECRET`) in env / `Envs`
- [x] `PvzHandoffTokenService` — sign/verify, `typ: 'pvz_handoff'`, TTL 30m, `jti`; verify checks Redis active `jti`
- [x] Redis on `POST /token`: `SET pvz:handoff:active:{clientId} {jti} EX 1800` (overwrites → invalidates previous QR)
- [x] Response DTO: `handoff_token`, `expires_at`, `pending_pickups_count` (no `qr_url`)
- [x] Error codes: `PVZ_HANDOFF_TOKEN_INVALID`, `PVZ_HANDOFF_TOKEN_EXPIRED`, `PVZ_HANDOFF_NO_PICKUPS`, `PVZ_HANDOFF_DELIVERY_NOT_ELIGIBLE`
- [x] Module `client/pvz-handoff` — controller, service, `api.decorators.ts`
- [x] `POST /client/pvz-handoff/token` — only authenticated client; check for pending `at_pickup_point`
- [x] `ClientPvzHandoffModule` registered in `AppModule`

### Backend: employee resolve & confirm

- [x] SQL `getClientDeliveriesAtEmployeeWarehouseSql` in `warehouse-ops.sql` + `npm run sql:generate`
- [x] `POST /employee/warehouse-ops/pvz-handoff/resolve` — verify token, warehouse filter, response DTO
- [x] `POST /employee/warehouse-ops/pvz-handoff/confirm` — `{ delivery_ids[] }`, reuse `transition` per id
- [x] `client_hint` helper (masked phone last 4 digits) for employee UI only
- [x] `POST /employee/warehouse-ops/pvz-handoff/client-products` — paginated line items (`product_id`, `product_title`, `product_price`, `order_id`, `quantity`, `product_image`) for client at employee PVZ
- [x] `PvzHandoffModule` imported in `EmployeeModule`

### Backend: tests

- [x] Unit: token service (valid, expired, wrong typ) — via `warehouse-ops.service.spec` + `client-pvz-handoff.service.spec`
- [x] Integration: token → resolve → client-products → confirm → `complete-delivery` (`marketplace-order-lifecycle.integration.spec.ts`)
- [ ] Integration: second `POST /token` → first token fails resolve with `PVZ_HANDOFF_TOKEN_INVALID`
- [ ] Integration: wrong warehouse employee → empty or forbidden
- [ ] Integration: confirm with alien `delivery_id` → error

### Frontend

- [ ] `marketplace-spa` — `POST /token` → `QRCode value={handoff_token}` (client-side only, e.g. qrcode.react)
- [ ] `marketplace-employee-app` — scan QR mode + list deliveries + confirm button + client-products list
- [ ] Locales ru/tj for static labels (if UI strings added)

## Phase 4.1 — Hardening & UX

- [x] Redis: one-time `jti` on resolve (replay protection via `pvz:handoff:used:{jti}`)
- [ ] `GET /client/pvz-handoff/session` — post-resolve state for client app
- [ ] Notification on resolve (in-app / push)
- [ ] Rate limit on employee `resolve`
- [ ] Audit: log `employee_id`, `client_id`, `delivery_ids` on confirm

## Docs & cross-links

- [ ] Link from [marketplace-merchants-and-warehouse-network.md](./marketplace-merchants-and-warehouse-network.md) «Still open» → this plan
- [ ] Update «В первом релизе вместо QR» note when phase 4.0 ships

## Manual QA checklist

- [ ] Client with 0 packages at PVZ — token endpoint behavior (404 vs empty token message)
- [ ] Client with 2+ `at_pickup_point` at same PVZ — resolve shows all; partial confirm
- [ ] Client at PVZ A, employee at PVZ B — resolve returns empty / forbidden
- [ ] Expired QR (>30m) — clear error on scan
- [ ] Re-issue QR on client — old screenshot/photo fails resolve
- [ ] Fallback: `external_order_id` lookup still works for single package
- [ ] After confirm: client `complete-delivery` per order → `completed`
