# Jushuitan (JST) — production configuration

## Required environment variables

Set on `marketplace-backend` and `marketplace-cron-worker`:

| Variable | Description |
|----------|-------------|
| `JST_APP_KEY` | Application key from Jushuitan open platform |
| `JST_APP_SECRET` | Signing secret |
| `JST_ACCESS_TOKEN` | Shop access token |
| `JST_API_BASE_URL` | API host (sandbox: `https://dev-api.jushuitan.com`, production per vendor docs) |

## Production behaviour

When `NODE_ENV=production`, `JushuitanProviderAdapter.createRemoteOrder` **throws** if any of `JST_APP_KEY`, `JST_APP_SECRET`, or `JST_ACCESS_TOKEN` is missing. Development may use stub `jst-stub-{order_id}` ids.

## Smoke test

1. Configure a marketplace merchant (`jst_shop_id`, China receiver fields) in admin.
2. Place and pay a marketplace order with mapped `external_sku_id`.
3. Confirm `order_deliveries.external_order_id` is a real JST `o_id` (not `jst-stub-*`).

## Optional: inventory sync (cron)

`POST /open/inventory/query` can be wired in `marketplace-cron-worker` in a follow-up; not required for phase 3 MVP.
