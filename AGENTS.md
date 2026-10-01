# AGENTS.md — Project Instructions for AI Coding Agents

These instructions apply to any AI agent working in this repository (Claude, Copilot, Antigravity, etc.) and take precedence over any generic/ambient AGENTS.md found outside this repository. `AGENTES.md` (repo root) records the history of the one big decision this file already bakes in — the Azure stack — plus the project's domain glossary; read it for that context, not for competing rules.

If project-specific instructions exist elsewhere in `docs/`, follow them as the detailed source of truth; this file stays short and points to them rather than duplicating.

---

## 1) Goals

You are a modern Web/SaaS engineer focused on production-quality code, maintainable architecture, security by default, mobile-first delivery, and fast iteration without over-engineering.

Tanqueo is used at the gas pump on a phone, by 2–4 people sharing a household budget of ≤ $5 USD/month in running cost (NFR-06). Every decision should protect that: fast on a phone, cheap to run, simple enough that a side project stays maintainable by one person.

Prefer clarity over cleverness. Prefer proven solutions.

---

## 2) Default Stack

Confirmed for this project (see [`docs/architecture/architecture.md`](docs/architecture/architecture.md) and [`ADR-0001`](docs/adr/ADR-0001-azure-static-web-apps-hosting.md)). This is **not** the generic Next.js/Supabase template some ambient AGENTS.md files default to — do not "correct" it back to that without a new ADR.

### Core Platform

- Hosting/API: Azure Static Web Apps with managed Functions (Node, TypeScript, HTTP triggers) — REST-first, **not** Next.js Route Handlers.
- Auth: Azure Static Web Apps built-in auth (Microsoft, GitHub identity providers) — not Supabase Auth.
- Database: Azure Database for PostgreSQL Flexible Server, accessed through one `db.ts` module (NFR-10).
- Authorization: enforced server-side in the API layer — every handler derives `household_id` from the caller's verified membership and filters every query by it (NFR-01). PostgreSQL RLS is **not** currently used; this is a recorded, unresolved gap — see [`ADR-0003`](docs/adr/ADR-0003-household-scoped-tenancy-without-database-rls.md) before assuming RLS exists.
- Files: Azure Blob Storage, private container, short-lived single-blob SAS URLs (NFR-02).
- Extraction: Azure OpenAI vision model with structured outputs, behind the `ReceiptExtractor` interface (see [`ADR-0002`](docs/adr/ADR-0002-vision-based-receipt-extraction-interface.md)).

### Frontend

- React + Vite + TypeScript — not Next.js App Router.
- **UI components: HeroUI v3 (`@heroui/react` + `@heroui/styles`), theme "Rabbit" — the project's one and only component library** (confirmed 2026-09-30, see [`ADR-0004`](docs/adr/ADR-0004-heroui-as-exclusive-component-library.md)). Built on Tailwind CSS v4 + React Aria Components; no provider needed; compound component API (`Card.Header`, not a `title` prop). **shadcn/ui does not apply to this project** — that was an earlier, now-superseded assumption. None of this is installed in `package.json` yet.
- Forms: React Hook Form + Zod.
- Client-only state: Zustand.
- PWA: `vite-plugin-pwa`, planned for Milestone M6 (FR-24) — not yet a dependency.

### Testing

- Unit/component: Vitest.
- E2E: Playwright.
- Neither is installed yet (`package.json` has no test runner and no `pnpm test` script). Add them at, or before, Milestone M1 — do not ship features without a way to run tests for them.

---

## 3) Non-Negotiable Rules

### Security

- Client input is untrusted.
- Authorization must live server-side, in the API layer, for every handler (see Authorization above — RLS is not yet the mechanism here, unlike a generic Supabase default).
- Default to deny when unsure.
- Never commit secrets: DB connection strings, Azure OpenAI keys, SAS-signing keys, or any credential. Keep them in Azure Static Web Apps application settings or ignored local environment files; `.env.example` may list variable names with empty values only.
- Run a secret scan and review the diff before committing.

### Validation

- Validate all external input server-side with Zod, regardless of which Function handles it.
- Accept unknown input as `unknown`, never trust the shape.
- Return clear, actionable errors without leaking internals (no stack traces, no raw DB errors, no provider responses).

---

## 4) API Policy (REST First)

- REST via Azure Functions HTTP triggers under `/api` — see the endpoint table in [`docs/architecture/architecture.md`](docs/architecture/architecture.md#core-modules) for the current surface.
- Every endpoint requires sign-in and resolves `household_id` from the caller — never from the request.
- Use proper HTTP status codes (`400` malformed, `401` unauthenticated, `403` unauthorized, `404` not visible/found, `409` conflicts, `422` semantically invalid, `500`/`503` server/dependency failure).
- `[Open question: no API versioning scheme exists yet — decide before the API is exposed beyond this household.]`

---

## 5) Database & Data Patterns

- Include `created_at` (and `updated_at` where a row is mutated after creation) timestamps.
- Scope every household-owned table by `household_id` — this project's tenant boundary is the **household**, not the individual user (contrast with a generic per-`user_id` SaaS default).
- Enable PostgreSQL RLS on household-owned tables as defense-in-depth once `ADR-0003` is resolved; until then, do not assume it is protecting anything.
- Prefer constraints and transactions over app-logic checks (e.g., unique plate per household, unique `(auth_provider, auth_user_id)`, `NOT NULL` where the schema in `docs/architecture/architecture.md` specifies it).
- Never run a migration that can delete or silently alter existing records without explicit user confirmation — this applies to every environment, including local dev once a database is provisioned.

---

## 6) Project Structure

Defined in [`docs/architecture/project-structure.md`](docs/architecture/project-structure.md) — read it before creating new files or directories. In short: `src/components/{app,commons,features/<domain>}`, feature-local `hooks/`, and a separate `api/src/functions/` per resource (not `src/app/api`, which is a Next.js convention this project doesn't use).

Follow existing repo conventions first; keep domains cohesive; shared code must be truly shared (promote to `commons/`/`src/hooks/` only on a second use-site, per the extraction thresholds in `docs/architecture/project-structure.md`).

---

## 7) Code Quality

- TypeScript strict where possible; avoid `any`.
- Shared types between web and API (NFR-10).
- Explain *why* in comments, not *what* — only when the reason isn't obvious from the code itself.

---

## 8) Configuration & Limits

Centralize limits in a single config module rather than scattering magic numbers. The following are already decided by the PRD/architecture and must stay in sync with it, not redefined ad hoc:

| Limit | Value | Rationale |
| --- | --- | --- |
| Receipt upload size | max 10 MB | FR-08 |
| Client-side resize | max 2000 px | FR-08, keeps upload/extraction fast |
| SAS URL validity | ≤ 10 minutes, single blob | NFR-02 |
| Extraction timeout | 20 s | FR-12 — beyond this, fall back to manual entry |
| Litres × price tolerance | ± $0.05 | FR-15 |
| High-consumption threshold | baseline × 1.15, needs ≥ 3 prior full-tank values | ADR/architecture invariants |
| Minimum distance for consumption | 50 km | Architecture invariants |
| Monthly budget alert | $5 USD | NFR-06 |

Document the rationale for any new limit the same way.

---

## 9) Error Handling & UX

- Always show loading states.
- Prevent double submissions (especially on save/upload, where a duplicate fill-up or double-billed extraction call is a real risk).
- Do not leak sensitive data in errors.
- A failed or low-confidence extraction must never block saving a fill-up — degrade to manual entry (FR-12, NFR-08); this is the concrete instance of "fail open on non-critical paths, fail closed on authorization."

---

## 10) Testing Expectations

- Cover critical logic first once test tooling exists: the consumption calculation, the duplicate-detection rule, and — highest priority given the ADR-0003 gap — cross-household isolation (no query should ever return another household's data).
- Do not mock the database (Azure PostgreSQL) in E2E tests.
- Place tests in the nearest `tests/` folder to the code they cover, per `docs/architecture/project-structure.md`.

---

## 11) Tooling & Delegation (Skills + MCP)

Full policy: [`docs/tooling-delegation.md`](docs/tooling-delegation.md). Summary:

### Source of truth

- Skills: `.agents/skills/` (tracked in `skills-lock.json`; installed/updated via `npx skills install` / `npx skills add`).
- MCP servers/config: none configured in this repository yet.

### Discovery protocol

Before implementing a non-trivial piece of work:

1. Check `.agents/skills/` for a relevant skill (e.g., `documentation-system-builder` for docs work, `vercel-react-best-practices` for React/Vite performance questions, `heroui-react` for any HeroUI component/theming work — fetch its docs instead of relying on memorized v2 patterns, which do not apply to this project's v3).
2. If an MCP exists for the system involved (e.g., a future Azure MCP), use it to confirm ground truth — schema, existing configuration, deployed state — before assuming.
3. Only proceed without a skill/MCP if neither exists for the task.

### Azure policy

There is no Supabase in this project. Prefer the documented decisions in `docs/architecture/architecture.md` over improvising Azure resource configuration; when an authoritative tool (Azure CLI, an MCP) is available, use it to confirm real schema/config state rather than assuming the docs are still accurate.

---

## 12) Working Process

1. Identify impacted areas.
2. Propose a minimal viable approach consistent with `docs/architecture/architecture.md` and `docs/architecture/project-structure.md`.
3. Implement end-to-end.
4. Run lint, typecheck, and tests (once test tooling exists).
5. Update the relevant file(s) under `docs/` in the same change whenever governed behavior changes — see [`docs/README.md`](docs/README.md) for which document owns which topic.

---

## 13) Git & Commits

- Write commit messages in English, Conventional Commits format, imperative tense, ≤ 72 characters on the summary line, no trailing period.
- Use this repo's feature areas as scopes where it helps: `auth`, `household`, `cars`, `receipts`, `fill-ups`, `dashboard`, `docs`.
- The `caveman-commit` skill is available in this environment for drafting a compressed, intent-only Conventional Commits message — use it via `/commit` or `/caveman-commit` when asked to write one.

Examples:

- `feat(receipts): add SAS upload endpoint`
- `fix(fill-ups): reject odometer below previous reading`
- `docs(architecture): record ADR-0003 tenancy gap`

---

## 14) Conflict Resolution

- Prefer existing repo conventions (this file, `docs/`) over an ambient or generic AGENTS.md found in a parent directory outside this repository.
- Security rules override convenience, every time.
- If this file and a `docs/` document disagree, `docs/` wins for its owned topic (see `docs/README.md`'s source-of-truth table) — fix this file instead of the doc.

---

## 15) What Not To Do

- Do not add Next.js, Supabase, Prisma, or any other framework/service from a generic default "just in case" — this project's stack is Azure-based by explicit decision (ADR-0001); a change requires a new ADR.
- Do not build a custom/bespoke UI component, or reach for shadcn/ui, plain styled `div`s, or another component library — HeroUI v3 is the only component library (ADR-0004). Check HeroUI's component set first; a bespoke component needs a stated, specific reason it's the only option.
- Do not use `onClick` on a HeroUI interactive component when `onPress` is available — it's there for accessibility, not style.
- Do not implement a screen against guessed "Rabbit" theme color values — confirm the actual theme tokens (built via HeroUI's Theme Builder) before writing UI that depends on them (ADR-0004).
- Do not bypass the household-scoping check in an API handler, and do not assume PostgreSQL RLS is protecting a table it isn't (ADR-0003).
- Do not invent abstractions without a clear, current benefit (no premature components, hooks, or config layers for a single use-site).
- Do not treat a partial fill-up as a full tank in any consumption calculation, and do not skip the litres × price mismatch warning (FR-15).
- Do not commit Azure secrets, connection strings, or SAS-signing keys, and do not run a destructive database migration without explicit confirmation.
