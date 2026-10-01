# Tanqueo

A responsive web app (PWA) that turns a photo of a gas receipt into a fuel log and a simple per-car dashboard for a household in Ontario, Canada.

Log a fill-up in under 30 seconds — photo, review, save — and see monthly spend, real consumption (L/100 km), average price (¢/L), cost per km, and spend by station, shared across a household of 2–4 people.

**Status (2026-09-29): pre-implementation.** The application is currently the default Vite + React scaffold; no Tanqueo features are built yet. Product requirements and architecture are fully documented and ready to implement — see [`docs/`](docs/README.md).

## Stack

React + Vite + TypeScript PWA on Azure Static Web Apps, with managed Functions as the API, PostgreSQL Flexible Server, Blob Storage, and Azure OpenAI for receipt extraction. See [`docs/architecture/architecture.md`](docs/architecture/architecture.md) for the full design and [`ADR-0001`](docs/adr/ADR-0001-azure-static-web-apps-hosting.md) for why.

## Getting Started

```bash
pnpm install
npx skills install   # installs agent skills tracked in skills-lock.json
pnpm dev
```

Other commands: `pnpm build`, `pnpm lint`, `pnpm preview`. See [`docs/contributing/CONTRIBUTING.md`](docs/contributing/CONTRIBUTING.md) for the full contribution workflow.

## Documentation

Start at [`docs/README.md`](docs/README.md) for the full index: product requirements, architecture, design, privacy, tooling delegation, ADRs, and runbooks. Repository-specific agent guidance lives in [`AGENTES.md`](AGENTES.md).
