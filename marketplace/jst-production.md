# Jushuitan (JST) — production configuration

## Required environment variables

Set in deployment `../.env` (passed to **backend** and **cron-worker** via `docker-compose.prod.yml`):

| Variable | Backend | Cron-worker | Description |
|----------|---------|---------------|-------------|
| `JST_APP_KEY` | yes | yes | Application key from Jushuitan open platform |
| `JST_APP_SECRET` | yes | yes | Signing secret |
| `JST_ACCESS_TOKEN` | yes | yes | Shop access token |
| `JST_API_BASE_URL` | yes | yes | Sandbox: `https://dev-api.jushuitan.com`; production URL from vendor docs |
| `JST_CATALOG_SYNC_ENABLED` | — | yes | `true` to run `JushuitanCatalogSyncJob` every 5 minutes |
| `JST_CATALOG_SYNC_RUN_ON_START` | — | yes | `true` — one sync when container starts |
| `JST_CATALOG_SYNC_LOOKBACK_DAYS` | — | yes | Incremental window (max 7 days per JST API), default `7` |
| `JST_SYNC_MERCHANT_SLUG` | — | yes | Target marketplace merchant slug, default `jushuitan-marketplace` |
| `JST_CN_WAREHOUSE_*` | — | optional | CN warehouse fields when sync auto-creates merchant |

## Production behaviour

When `NODE_ENV=production`, `JushuitanProviderAdapter.createRemoteOrder` **throws** if any of `JST_APP_KEY`, `JST_APP_SECRET`, or `JST_ACCESS_TOKEN` is missing. Development may use stub `jst-stub-{order_id}` ids.

## Smoke test

1. Configure a marketplace merchant (`jst_shop_id`, China receiver fields) in admin.
2. Place and pay a marketplace order with mapped `external_sku_id`.
3. Confirm `order_deliveries.external_order_id` is a real JST `o_id` (not `jst-stub-*`).

## Optional: inventory sync (cron)

`POST /open/inventory/query` can be wired in `marketplace-cron-worker` in a follow-up; not required for phase 3 MVP.
