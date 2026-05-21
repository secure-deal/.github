# Marketplace merchants & warehouse network — progress

> Last updated after phase 2 implementation (backend + minimal SPA/employee apps).

## Phase 0 / foundations (done)

- [x] `merchants.type` (`local` | `marketplace`)
- [x] Admin CRUD `pickup_points`
- [x] `merchant_warehouses` (local merchant warehouses)
- [x] Jushuitan DB fields + `marketplace_product_mappings`
- [x] Cron Jushuitan catalog sync (`marketplace-cron-worker`)
- [x] Admin merchant `type` on create/edit (partial marketplace config fields in DB)

## Phase 1 prerequisites (done in this iteration)

- [x] Migration `1779800000000-marketplace-warehouse-network.psql` — `warehouses`, extended `order_deliveries`, `orders.pickup_point_id`, cargo tariffs, status events
- [x] `DeliveryStatusService` (`local` / `marketplace` transitions + order status mapping)
- [x] `MarketplaceFulfillmentService` + `JushuitanProviderAdapter` + `createRemoteOrder` after paid checkout
- [x] Admin `platform-warehouses` CRUD API
- [x] Admin merchants filter `merchant_type`
- [x] `operators` HTTP alias → `admin/employees`

## Phase 2 (stages 7–13) — done (backend + MVP UI)

### Stage 7 — Employees & mobile API

- [x] `GET /employee/warehouse-ops/orders/lookup?external_order_id=`
- [x] `POST /employee/warehouse-ops/deliveries/:id/transition`
- [x] `GET /employee/warehouse-ops/deliveries` (queue at employee warehouse)
- [x] `admins.warehouse_id` binding

### Stage 8 — Client catalog & checkout

- [x] `GET /client/pickup-points` (city, near_lat/lng, limit)
- [x] Checkout `pickup_point_id` for marketplace cart (no `delivery_address_id`)
- [x] Catalog `is_international`, `estimated_delivery_days_min`, `origin_country` from product attributes
- [x] SPA page `marketplace-spa/src/app/checkout/pickup/page.tsx` (list + select PVZ)

### Stage 9 — Client timeline

- [x] `order_delivery_status_events` table (backend)
- [ ] Client order API `delivery_timeline[]` in response (extend `findByIdForClient` — follow-up)

### Stage 10–11 — Employee mobile app

- [x] `marketplace-employee-app` — scan/lookup/transition UI
- [ ] Admin order card: marketplace source + PVZ route display (admin-panel follow-up)

### Stage 12 — Cargo & settings

- [x] `marketplace_cargo_tariffs` table + checkout fee calculation
- [x] `MARKETPLACE_ORDER_MARKUP_PERCENT` in settings registry (default 5)
- [x] Order snapshot: `cargo_fee`, `marketplace_markup_percent`, `marketplace_service_fee`

### Stage 13 — Provider extension point

- [x] `MarketplaceProviderAdapter` interface + `MarketplaceProviderRegistry`
- [x] `JushuitanProviderAdapter` with dev stub when credentials missing
- [x] Disputes blocked for `merchants.type = marketplace` (SQL guard)

## Phase 3 (productization) — done

- [x] Seed platform warehouses + `pickup_points.warehouse_id` + default cargo tariff (`1779900000000-seed-platform-warehouses.psql`)
- [x] Admin API: marketplace config, cargo tariffs, order detail marketplace block + timeline
- [x] Admin-panel: merchant type filter, JST/cargo UI, platform warehouses, order card, employees nav
- [x] Client `delivery_timeline[]` + SPA order page + Yandex Maps on pickup (with API key)
- [x] Employee-app: login, queue, scan transitions
- [x] JST production guard + `docs/marketplace/jst-production.md`

## Still open / follow-up (phase 4+)

- [ ] Shared npm package for JST client (cron + backend dedup)
- [ ] Admin: `POST open/shops/query` for `jst_shop_id` resolve tool
- [ ] Cron optional inventory sync

## Jushuitan integration checklist

- [x] MVP `provider_code=jushuitan`
- [x] Cron `POST /open/sku/query`
- [x] `createRemoteOrder` → `POST /open/jushuitan/orders/upload` (backend adapter)
- [ ] Admin: `POST open/shops/query` for `jst_shop_id`
- [ ] Cron optional inventory sync
