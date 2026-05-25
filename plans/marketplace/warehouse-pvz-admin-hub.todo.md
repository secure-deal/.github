# warehouse-pvz-admin-hub — чеклист реализации

Основной план: [warehouse-pvz-admin-hub.md](./warehouse-pvz-admin-hub.md)

## Фаза 1 — БД и backend

- [ ] `1780400000000-warehouse-hub-audit.psql` (actor `employee`, view `v_warehouse_fulfillment_lines`)
- [ ] `warehouse-hub` module: SQL, service, controller, DTOs
- [ ] `GET /admin/warehouse-hub/:warehouseId` (+ employees, products, orders)
- [ ] `GET /admin/employees/:id/audit-logs`
- [ ] `warehouse_id` filter на `GET /admin/employees`
- [ ] Audit writes в `WarehouseOpsService`
- [ ] `npm run sql:generate` + backend tests

## Фаза 2 — Admin panel

- [ ] `admin-warehouse-hub-api.ts` + hooks
- [ ] `/platform-warehouses/[id]` hub (tabs)
- [ ] `/pickup-points/[slug]` hub (tabs + settings)
- [ ] `/employees/[id]` — журнал audit
- [ ] Links с list pages + ru/tj locales

## Фаза 3 — Seeder

- [ ] merchants → `warehouses` (не `merchant_warehouses`)
- [ ] `platform-warehouses.seeder.ts`, `pickup-points.seeder.ts`, `employees.seeder.ts`
- [ ] orders: marketplace + pickup_point_id / warehouse statuses
- [ ] `audit-log.seeder.ts`, disputes → employee operators
- [ ] Прогон `seed:secure-deal`

## Фаза 4 — Полировка

- [ ] `generate:api:local` (опционально)
- [ ] Smoke QA hub в браузере
