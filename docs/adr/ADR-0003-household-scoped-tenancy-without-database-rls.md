# ADR 0003 — Enforce household tenancy in application code, not database RLS

## Status

Proposed — the source PRD makes this decision implicitly (NFR-01), but it has not been explicitly weighed against the ambient security policy until this documentation pass. Treat as Proposed until the open question below is resolved.

## Context

Every table except `users` and `vehicle_ratings` is scoped to a household (`household_id`). The PRD's stated rule (NFR-01) is: "the API derives it from the caller's membership, never from the request. Every query filters by it." The ambient engineering `AGENTS.md` (ADR-0001's context) states as a non-negotiable rule: "Authorization must live server-side and in RLS," with PostgreSQL RLS as the default authorization mechanism. Tanqueo's PRD does not mention RLS anywhere, and its "Definition of done" instead relies on testing ("No API call can read or write another household's data (tested)").

## Decision

For MVP, enforce tenancy exclusively in the API layer: every Function handler resolves `household_id` from the authenticated caller's `household_members` row and includes it in every query's `WHERE` clause. No PostgreSQL Row-Level Security policies are defined. Isolation is verified by tests rather than by a database-enforced invariant.

## Alternatives Considered

- **PostgreSQL RLS as defense-in-depth**, matching the ambient `AGENTS.md` default: define a policy per table keyed on `household_id`, set via `SET LOCAL` or a session variable per request. Not adopted in the source PRD, and not retroactively added by this documentation pass, since that is a product/architecture decision this skill is not authorized to make on its own (`documentation-system-builder`'s role: "Do not implement application features; inspect code only to extract evidence").
- **A shared ORM-level scoping helper** (e.g., a query builder that always injects `household_id`) as a lighter-weight middle ground. Not evaluated in the source PRD.

## Consequences

- Positive: a single relational schema with straightforward queries; no additional Postgres session-variable plumbing needed before Milestone M1.
- Negative: no defense-in-depth — a single handler that forgets to filter by `household_id` is a cross-household data leak with nothing else to catch it. This is the highest-severity gap this documentation pass surfaced.
- `[Open question: decide whether to add PostgreSQL RLS before Milestone M1 (Foundations) ships, keep the app-layer-only model and rely on the PRD's "tested" isolation requirement, or document a compensating control (e.g., an integration test suite that runs against every endpoint with two households and asserts isolation, gating CI). This decision should update this ADR's Status to Accepted once made, and should be reflected back into `docs/prd/PRD-v1.md` §8 (NFR-01) if it changes the requirement's wording.]`
