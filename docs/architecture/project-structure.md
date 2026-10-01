# Project Structure

This is the canonical structure document referenced by the ambient engineering `AGENTS.md` ("Canonical Project Structure (Default)"). It covers the repository as it exists today and the target layout once implementation starts.

## Current Repository (as of 2026-09-29)

```text
tanqueo-web-app/
├── .agents/skills/                 # installed agent skills (documentation-system-builder, vercel-react-best-practices)
├── .claude/skills/                 # symlinks into .agents/skills for Claude Code
├── docs/                           # this documentation system
├── public/
│   ├── favicon.svg
│   └── icons.svg
├── src/
│   ├── assets/                     # hero.png, react.svg, vite.svg
│   ├── App.tsx                     # [Vite default scaffold — not yet replaced with Tanqueo UI]
│   ├── App.css
│   ├── index.css
│   └── main.tsx                    # React root mount
├── index.html                      # Vite HTML entry
├── eslint.config.js
├── vite.config.ts
├── tsconfig.json / tsconfig.app.json / tsconfig.node.json
├── package.json
├── pnpm-lock.yaml
├── skills-lock.json
└── Tanqueo — Product Requirements Document.md   # superseded by docs/prd/PRD-v1.md
```

[Inferred from codebase: directory listing and `package.json` as of this migration.] There is no `src/app`, `api/`, `components/`, or `hooks/` yet — none of the PRD's features are implemented. `package.json` dependencies are limited to `react`, `react-dom`, and the default Vite/ESLint/TypeScript toolchain; no Tailwind, HeroUI, React Hook Form, Zod, Zustand, Vitest, or Playwright are installed yet, despite `AGENTS.md` naming them as the default stack.

The root `Tanqueo — Product Requirements Document.md` should be removed once this migration is approved, since `docs/prd/PRD-v1.md` is now its source of truth.

## Target Structure

Combines the PRD's confirmed Azure architecture (see `docs/architecture/architecture.md`) with the ambient `AGENTS.md`'s framework-agnostic conventions (component placement, naming, barrel files, testing). The Next.js-specific parts of `AGENTS.md` (`src/app/api/v{major}/...` Route Handlers) do not apply — see `AGENTES.md`.

```text
tanqueo-web-app/
├── src/                             # React PWA (Vite), deployed via Azure Static Web Apps
│   ├── components/
│   │   ├── app/                     # layout/structural: header, active-car switcher, tab nav
│   │   ├── commons/                 # generic UI: empty states, confidence pill, full/partial badge
│   │   └── features/
│   │       ├── cars/                # add/edit car, NRCan lookup UI
│   │       ├── receipts/            # capture, upload, review form
│   │       ├── fill-ups/            # save flow, duplicate sheet, history list
│   │       ├── dashboard/           # per-car dashboard widgets
│   │       └── household/           # invite flow, member list
│   ├── hooks/                       # cross-feature hooks only, promoted from a feature's local hooks/
│   ├── assets/
│   ├── App.tsx
│   └── main.tsx
├── api/                             # Azure Functions HTTP API (Node/TypeScript) — NOT Next.js Route Handlers
│   └── src/
│       ├── functions/               # one file per resource: me, cars, receipts, fill-ups, invites, vehicle-ratings, dashboard
│       ├── extraction/              # ReceiptExtractor interface + AzureOpenAIReceiptExtractor
│       └── db.ts                    # single DB-access module (NFR-10)
├── db/
│   └── migrations/                  # schema migrations for the 8-table model in architecture.md
├── docs/                            # this documentation system
└── public/
```

### UI components — HeroUI only

Per [`ADR-0004`](../adr/ADR-0004-heroui-as-exclusive-component-library.md), HeroUI (`@heroui/react`, theme "Rabbit") is the project's one and only component library. This changes what "new code" in `components/` means:

- `components/app/`, `components/commons/`, and `components/features/<domain>/` hold **compositions of HeroUI components** (e.g. a `fill-up-card.tsx` that arranges HeroUI `Card`, `Chip`, and `Button` for this app's data) — not new visual primitives built from styled `div`s.
- Before writing any new component, check HeroUI's component set first. Reach for a custom/bespoke component only when a specific case genuinely cannot be solved by composing HeroUI components — and say so (PR description or a short code comment) when that happens.
- `src/components/ui/` (a bespoke primitives folder) does not exist in this project and should not be created — that role belongs to `@heroui/react` itself.

### Where new code belongs

- A feature maps 1:1 to a PRD functional area (cars, receipts, fill-ups, dashboard, household). Start new UI in `src/components/features/<domain>/`, composed from HeroUI components.
- Promote a composed component to `components/commons/` only on its second use-site across features (per `AGENTS.md`'s "Abstraction threshold" rule — unchanged by the stack override).
- Promote a hook to `src/hooks/` only once it is reused across two different features; otherwise keep it in the feature's own `hooks/` folder next to the component that owns it.
- New API resources go under `api/src/functions/`, one file per resource, matching the endpoint table in `docs/architecture/architecture.md`. Apply Zod validation and the app-layer tenancy check (`household_id` from caller identity, never from the request) at the top of every handler.
- Database access goes only through `api/src/db.ts` (NFR-10) — no ad hoc connections elsewhere.

### Conventions that still apply unchanged from `AGENTS.md`

These are framework-agnostic and hold regardless of the Azure/Vite vs. Next.js/Supabase decision:

- Kebab-case file names (`use-active-car.ts`, `fill-up-card.tsx`); PascalCase components; camelCase hooks/utilities; `SCREAMING_SNAKE_CASE` constants.
- Barrel files (`index.ts`) once a folder exports more than two public components/utilities.
- `data-testid` on every application-owned component's root element, named by rendered role (e.g. `fill-up-history-list`, not `card`).
- Inline component/hook extraction thresholds (~40–50 lines of JSX, 3+ `useEffect`s, 2+ inline async calls) and placement rules (`commons/` vs `features/<domain>/`).
- Test files live in the nearest `tests/` folder to the code they cover (e.g. `src/components/features/fill-ups/tests/`), not loose next to the component.

### Ownership

`[Assumed: solo-maintained household side project — no team ownership map exists today. Update this section if collaborators join.]`
