# SmartPick — Electronics Advisor

SmartPick helps users find the best electronics (phones, laptops, headphones, cameras, etc.) based on their budget, use case, and preferences — then shows exactly where to buy them online or in-store.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/smartpick run dev` — run the frontend (port varies)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite + Tailwind CSS + shadcn/ui + wouter + TanStack Query
- API: Express 5
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `lib/api-spec/openapi.yaml` — OpenAPI spec (source of truth)
- `lib/api-client-react/src/generated/` — React Query hooks (auto-generated)
- `lib/api-zod/src/generated/` — Zod validation schemas (auto-generated)
- `lib/db/src/schema/recommendations.ts` — Recommendation session DB schema
- `artifacts/api-server/src/routes/recommendations.ts` — API routes
- `artifacts/api-server/src/lib/product-catalog.ts` — Curated product database (~20 products)
- `artifacts/api-server/src/lib/recommendation-engine.ts` — Rule-based scoring engine
- `artifacts/smartpick/src/pages/` — Frontend pages (Home, Wizard, Results, History)

## Architecture decisions

- **Rule-based recommendation engine** — scores products by budget fit (35pts), use-case match (30pts), feature preferences (15pts), brand preference (10pts), and review-weighted quality (10pts). No external AI API needed.
- **Product catalog in-memory** — curated list of ~20 real products with accurate specs, prices, and buy links. Seeded into the recommendations JSON column on each session.
- **Sessions stored in Postgres** — each recommendation run is persisted as a session (category, budget, use case, matched products JSON) so users can revisit history.
- **OpenAPI-first** — contract defined in `openapi.yaml`, codegen produces typed React Query hooks and Zod schemas automatically.

## Product

- **Homepage** — hero, category grid, trending products showcase
- **Wizard** — 4-step form: Category → Budget → Use Case → Brand/Feature Preferences
- **Results** — top pick + alternatives with match score, pros/cons, buy buttons (Amazon, Flipkart, Best Buy, offline stores)
- **History** — past recommendation sessions, click to revisit

## User preferences

_Populate as you build._

## Gotchas

- After any OpenAPI spec change, always re-run `pnpm --filter @workspace/api-spec run codegen` then `pnpm run typecheck:libs` before running leaf artifact typechecks
- `react-icons/si` brand icons are not reliably available — use Lucide icons instead
- Products are stored as JSONB in the `recommendation_sessions` table — no separate products table

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
