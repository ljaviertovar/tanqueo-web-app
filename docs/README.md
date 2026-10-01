# Documentation Index

Authoritative documentation for Tanqueo. Each topic below has exactly one source-of-truth document — if you find the same claim duplicated elsewhere, treat this index's target as correct and fix the duplicate.

## Index

| Topic | Document | Owns |
| --- | --- | --- |
| Product requirements | [`prd/PRD-v1.md`](prd/PRD-v1.md) | Problem, goals, users, user journeys, functional/non-functional requirements, roadmap |
| Architecture | [`architecture/architecture.md`](architecture/architecture.md) | System design, data model, API contract, security model, trade-offs |
| Project structure | [`architecture/project-structure.md`](architecture/project-structure.md) | Repository map, where new code belongs, naming conventions |
| Design | [`design/DESIGN.md`](design/DESIGN.md) | Product experience, responsive/accessibility targets, screen behavior |
| Privacy | [`privacy/taste-profile.md`](privacy/taste-profile.md) | Data collected, residency, retention, consent boundaries |
| Tooling & delegation | [`tooling-delegation.md`](tooling-delegation.md) | Skill registry, what agents may do without asking vs. what needs approval |
| Architecture decisions | [`adr/`](adr/) | Context, alternatives, and consequences for individual decisions |
| Runbooks | [`runbooks/`](runbooks/) | Deployment and incident-response procedures |
| Contributing | [`contributing/CONTRIBUTING.md`](contributing/CONTRIBUTING.md) | Setup, everyday commands, verification steps before a change |

Repository-specific agent guidance (including where this repo's stack diverges from generic engineering defaults) lives in [`AGENTES.md`](../AGENTES.md) at the repo root.

## Source of Truth Rules

- Product intent and requirements: `prd/`, never restated as fact elsewhere — other docs link to it.
- System design and technical decisions: `architecture/`.
- One current PRD version at a time under `prd/`; a new version is created only for a real product revision, never to "complete a pattern."
- An ADR exists only for an evidence-backed architectural decision — not to populate the directory.

## Status

As of 2026-09-29, this documentation set describes the confirmed product and architecture decisions for a repository that is still at the default Vite + React scaffold stage (pre-Milestone-M1). Several documents contain explicit `[Open question: ...]` markers — resolve those before treating the affected sections as final.
