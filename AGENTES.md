# AGENTES.md

Repository-specific guidance for agents working on Tanqueo. This file exists because a generic engineering template exists outside this repository and defaults to a stack that does not match this project's actual, confirmed architecture. Read this file alongside `AGENTS.md` (repo root, the project's own authoritative instructions): most of the generic template's spirit still applies; the parts listed below do not.

## Stack Override (confirmed 2026-09-29)

Tanqueo's product architecture, as documented in [`docs/prd/PRD-v1.md`](docs/prd/PRD-v1.md) and [`docs/architecture/architecture.md`](docs/architecture/architecture.md), is:

- **Hosting/API:** Azure Static Web Apps with managed Functions (Node/TypeScript, HTTP triggers) — not Next.js Route Handlers.
- **Auth:** Azure Static Web Apps built-in auth (Microsoft, GitHub) — not Supabase Auth / Google OAuth.
- **Database:** Azure Database for PostgreSQL Flexible Server, accessed through one `db.ts` module — not Prisma over Supabase.
- **Authorization:** enforced in application code (every handler derives and filters by `household_id`) — **not** PostgreSQL RLS. This is a known, currently unresolved gap against the generic security default; see [`ADR-0003`](docs/adr/ADR-0003-household-scoped-tenancy-without-database-rls.md).
- **Files:** Azure Blob Storage with SAS-scoped access — not a Supabase Storage bucket.
- **Frontend:** React + Vite + TypeScript PWA — not Next.js App Router.

See [`ADR-0001`](docs/adr/ADR-0001-azure-static-web-apps-hosting.md) for the full rationale. If this project later moves toward the generic template's defaults, that requires its own ADR and a new PRD version (`PRD-v2.md`) — it is not implied or pre-approved by anything in this file.

## UI Override (confirmed 2026-09-30)

**[HeroUI v3](https://heroui.com)** (`@heroui/react` + `@heroui/styles`), theme "Rabbit," is the project's one and only UI component library — not shadcn/ui, not plain Tailwind markup, not any other kit. See [`ADR-0004`](docs/adr/ADR-0004-heroui-as-exclusive-component-library.md).

- No custom or from-scratch components. Compose HeroUI's compound components (`Card.Header`, `Card.Title`, etc.) for every screen. A bespoke component is justified only when a specific case genuinely cannot be solved by any HeroUI component, and that reason should be stated where the component is introduced.
- HeroUI v3 is a rewrite from the older v2/NextUI lineage: **no provider**, compound component API, CSS-based theming (no `framer-motion`). Do not apply memorized v2 patterns — use this repo's `heroui-react` skill to fetch current v3 docs before implementing any component.
- "Rabbit" is a custom theme (built via HeroUI's Theme Builder), not a shipped preset — its exact token values were not yet defined as of 2026-09-30. Do not build a screen against guessed colors; confirm the real theme tokens first.

## What Still Applies Unchanged From the Generic Template

These are framework-agnostic and hold for Tanqueo as written:

- Validation: all external input validated server-side with Zod, regardless of whether the handler is a Next.js Route Handler or an Azure Function.
- Forms: React Hook Form + Zod, Zustand for client-only state — `[Assumed: no Tanqueo-specific override exists for these; none are installed in `package.json` yet.]`
- Testing: Vitest for unit/component, Playwright for E2E — not yet added to `package.json` (see `docs/contributing/CONTRIBUTING.md` open question).
- Naming conventions, barrel file policy, component reusability thresholds, inline component/hook extraction rules — see `docs/architecture/project-structure.md` for how they map onto this repo's actual folders (`components/`, `hooks/`, `api/` instead of `src/app/api`), now composed from HeroUI rather than shadcn/ui or bespoke primitives.
- `data-testid` requirements on application-owned components.
- Security non-negotiables other than the RLS specifics above: client input is untrusted, default to deny when unsure, never commit secrets, run a secret scan before committing.
- Error handling & UX rules: loading states, no double submissions, no sensitive data in errors.
- Documentation ownership: update the relevant `docs/` file in the same change whenever governed behavior changes.

## What Does Not Apply

- The entire **API Policy** section of the generic template (`/api/v{major}/...` under `src/app/api`, REST versioning headers, `Idempotency-Key`, OpenAPI-per-Next.js-convention). Tanqueo's API is a flat Azure Functions surface documented in `docs/architecture/architecture.md`'s Core Modules section. Apply the same underlying principles (validate everything, one predictable error shape, don't leak internals) through that structure instead.
- Any reference to Supabase-specific tooling (RLS policies as the only authorization mechanism, Supabase CLI, Supabase migrations).
- shadcn/ui, or any instruction to build/style bespoke UI primitives — see UI Override above.

## Domain Glossary

- **Household** — the single MVP tenant; 2–4 members sharing cars and fill-ups. Not a general multi-tenant concept.
- **Car** — belongs to a household; has an official NRCan consumption rating and a user-confirmed one.
- **Fill-up** — one fuel purchase event, tied to a car and optionally a receipt.
- **Full tank vs. partial** — whether the fill-up topped off the tank; consumption is only computed between two full tanks.
- **One-off** — a fill-up flagged to be excluded from consumption baselines and averages (e.g. an unusual trip).
- **Active car** — the per-user default car shown in the header, switchable at any time.

## Where to Look

Start at [`docs/README.md`](docs/README.md) for the full documentation index. This file only records what's specific to Tanqueo versus generic defaults — it is not a substitute for the documents it points to.
