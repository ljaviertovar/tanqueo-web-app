# Contributing

Human contribution setup and verification for Tanqueo, referenced by `AGENTS.md`.

## Setup

```bash
pnpm install
npx skills install   # installs agent skills tracked in skills-lock.json
```

Before starting any UI work, install HeroUI v3 (not yet in `package.json` — see [`ADR-0004`](../adr/ADR-0004-heroui-as-exclusive-component-library.md)):

```bash
pnpm add @heroui/react @heroui/styles
```

Then add `@import "@heroui/styles";` right after `@import "tailwindcss";` in `src/index.css` (Tailwind CSS v4 itself also needs installing first — neither is in `package.json` yet). No provider wrapper is needed in `src/main.tsx` — HeroUI v3 doesn't use one.

## Everyday Commands

Per `package.json` as of 2026-09-29:

| Command | Purpose |
| --- | --- |
| `pnpm dev` | Start the Vite dev server |
| `pnpm build` | Type-check (`tsc -b`) then build for production |
| `pnpm lint` | Run ESLint |
| `pnpm preview` | Preview a production build locally |

`[Open question: no test command exists yet. AGENTS.md mandates Vitest for unit/component tests and Playwright for E2E, but neither is a dependency in package.json yet. Add them and a pnpm test script at, or before, Milestone M1.]`

## Before Opening a Change

1. Run `pnpm lint` and `pnpm build` (type-check) and fix failures.
2. Run tests once test tooling exists (see open question above); until then, manually verify the change against the relevant PRD functional requirement(s) in `docs/prd/PRD-v1.md`.
3. If the change affects product behavior, architecture, or scope, update the corresponding document under `docs/` in the same change (`AGENTS.md`'s "Documentation Ownership" rule) rather than letting docs drift.

## Conventions

Follow `AGENTS.md` for naming, component placement, barrel files, and testing expectations — see `docs/architecture/project-structure.md` for how those apply to this repository's actual (Vite + Azure Functions + HeroUI) stack, and `AGENTES.md` for domain glossary and historical override context.

Build every screen by composing HeroUI v3 components — no custom/bespoke components except where HeroUI genuinely cannot solve a case (`ADR-0004`).

## Security

Never commit secrets (DB connection strings, Azure OpenAI keys, SAS signing keys). Keep them in Azure Static Web Apps application settings or local ignored environment files only; `.env.example` may list variable names with empty values. See `docs/tooling-delegation.md` for the full delegation and approval policy.
