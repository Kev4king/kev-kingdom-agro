# Kev Kingdom Agro Farm

A premium Pi Network-integrated farm marketplace for Kev Kingdom Agro Farm — livestock, aquaculture, and fresh produce ordered and paid with Pi cryptocurrency.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 5000)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string
- Optional env: `PI_API_KEY` — Pi Network server-side API key (for live payment approval/completion)

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite (served at `/`)
- API: Express 5 (served at `/api`)
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)
- Pi Network SDK v2.0 (loaded via CDN in index.html)

## Where things live

- `lib/api-spec/openapi.yaml` — API contract source of truth
- `lib/db/src/schema/orders.ts` — Orders table schema
- `artifacts/agro-farm/src/App.tsx` — Full storefront (product catalog, Pi SDK auth, checkout modal, payment flow)
- `artifacts/agro-farm/src/types/pi-sdk.d.ts` — TypeScript type declarations for Pi SDK
- `artifacts/api-server/src/routes/payments.ts` — Pi payment approve/complete endpoints
- `artifacts/api-server/src/routes/orders.ts` — Order CRUD endpoints

## Architecture decisions

- Pi SDK initialized at app load with `Pi.init({ version: "2.0", sandbox: true })` and `Pi.authenticate(['username', 'payments'])` — username shown in header nav.
- Payment flow: frontend calls `Pi.createPayment()` → `onReadyForServerApproval` fires → backend calls Pi API to approve → order saved to DB → `onReadyForServerCompletion` fires → backend calls Pi API to complete → order marked "paid".
- `PI_API_KEY` env var is optional; if missing, routes still return success (for sandbox/dev). Set it for production live payments.
- Orders stored in PostgreSQL with status tracking: `pending` → `paid`.
- All backend routes mounted at `/api` prefix; route handlers use relative paths (e.g., `/orders`, not `/api/orders`).

## Product

- **Storefront**: Premium agriculture marketplace with deep forest green (#1b4d3e) theme, Playfair Display serif headers, off-white off warm background.
- **Categories**: Livestock (Bull, Goat, Pig), Mini-Livestock (Grasscutter, Jumbo Snails), Aquaculture (Catfish, Tilapia), Crops (Plantain, Seasonal Produce).
- **Checkout**: Mobile pop-up modal collects Full Name, Phone, Delivery Address, State/Region before triggering Pi.createPayment().
- **Pi Auth**: Customer's Pi username displayed in the header nav bar after authentication.

## User preferences

- Use Pi Network SDK v2.0, sandbox: true
- Forest green (#1b4d3e) as the primary brand color
- Product categories: Livestock, Mini-Livestock, Aquaculture, Crops

## Gotchas

- The Pi SDK will show a postMessage error in regular browsers — this is expected. It works correctly inside Pi Browser.
- Set `PI_API_KEY` environment secret for live/mainnet payment approval. In sandbox mode, payments can be tested without it.
- Run `pnpm --filter @workspace/api-spec run codegen` after any changes to `lib/api-spec/openapi.yaml`.
- Run `pnpm --filter @workspace/db run push` after any changes to DB schema files.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
