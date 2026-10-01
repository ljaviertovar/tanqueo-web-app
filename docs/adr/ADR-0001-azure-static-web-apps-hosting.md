# ADR 0001 — Host Tanqueo on Azure Static Web Apps with managed Functions

## Status

Accepted (recorded in the source PRD, Sep 19, 2026; confirmed as the vigente architecture during documentation migration, 2026-09-29).

## Context

Tanqueo needs a frontend host, an API layer, a database, file storage, and a vision model for receipt extraction, all running within a ≤ $5 USD/month budget (NFR-06) for a single household (2–4 users). The ambient engineering `AGENTS.md` (a generic, cross-project template, not checked into this repository) defaults to Next.js + Supabase for new projects, but that decision was not made for Tanqueo specifically.

## Decision

Run the frontend as a React + Vite PWA on Azure Static Web Apps (Free tier), with its managed Functions (Node/TypeScript, HTTP triggers) as the API. Use Azure Database for PostgreSQL Flexible Server (Burstable B1ms) for relational data, Blob Storage (private container, SAS-scoped access) for receipt photos, and Azure OpenAI (vision model, structured outputs) for extraction. All resources run in Canada Central.

## Alternatives Considered

- **Next.js + Supabase (the ambient `AGENTS.md` default).** Would provide Route Handlers, Postgres with RLS, and Supabase Auth out of the box. Not chosen for this project's first version — the PRD was authored around Azure Static Web Apps' built-in auth (Microsoft/GitHub) and free-tier Functions, and no migration decision away from that has been made.
- **A dedicated Azure Function App from the start** (instead of SWA's managed Functions). Would support managed identity immediately, avoiding a DB password. Not chosen initially because SWA's managed Functions ship frontend and API together from one GitHub-connected resource at zero marginal cost; the managed-identity gap is accepted as a known trade-off with a documented upgrade path.

## Consequences

- Positive: frontend and API deploy together from GitHub with no extra infrastructure to provision; auth is zero-config; the whole stack fits the free-tier budget.
- Positive: the single DB-access module (`db.ts`, NFR-10) contains the blast radius of the one accepted trade-off below.
- Negative: SWA managed Functions do not support managed identity, so the API authenticates to PostgreSQL with a password (mitigated by TLS + Azure-services-only firewall, NFR-03). Documented upgrade path: SWA Standard + a linked Function App (Flex Consumption) with managed identity.
- Negative: this diverges from the ambient `AGENTS.md` default stack. `AGENTES.md` (repo root) records this as an explicit, scoped override so future contributors and agents don't silently "fix" the stack back to Next.js/Supabase.
