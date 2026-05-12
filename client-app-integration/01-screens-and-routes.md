# Screens and routes

This route map is based on the current buyer reference app in `app/bank-emulator/src/app`.

## Main routes

| Route | Purpose |
| --- | --- |
| `/` | App entrypoint / redirect shell. |
| `/login` | Client sign-in bootstrap. |
| `/home` | Main home/dashboard surface. |
| `/marketplace` | Product listing. |
| `/marketplace/search` | Search-focused catalog screen. |
| `/marketplace/[id]` | Product detail. |
| `/favorites` | Favorite products. |
| `/cart` | Current cart. |
| `/checkout` | Delivery address + checkout review. |
| `/checkout/otp` | Payment / OTP step. |
| `/checkout/success` | Post-checkout confirmation. |
| `/orders` | Buyer order list. |
| `/orders/[id]` | Buyer order detail. |
| `/disputes` | Buyer dispute list. |
| `/disputes/[id]` | Buyer dispute detail and chat. |
| `/notifications` | Notification inbox. |
| `/notifications/[id]` | Notification detail. |
| `/profile` | Buyer profile. |
| `/profile/setup` | Profile completion / setup flow. |

## Optional or demo-only routes

| Route | Notes |
| --- | --- |
| `/secure-deal` | Demo shell / reference section. |
| `/services` | Demo service screen, not core marketplace flow. |
| `/history` | Bank-emulator-specific history surface. |
| `/transfers` | Bank-emulator-specific transfer surface. |

## Routing rules

1. Use backend UUIDs for deep-link routes like `/orders/[id]`, `/disputes/[id]`, and notification detail.
2. Public display numbers like `order_id` should not replace the UUID route param.
3. Product detail is slug-based in the backend catalog contract even if the frontend route segment is named `[id]`.
4. If the buyer app diverges from bank-emulator IA, keep the backend resource mapping the same.
