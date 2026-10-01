# Changelog

All notable changes to this project are documented here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/).

## [Unreleased]

### Added

- Product Requirements Document v1 (`docs/prd/PRD-v1.md`), migrated and restructured from the original root-level PRD.
- Full documentation system: architecture, project structure, design, privacy, tooling delegation, three ADRs, deploy/incident runbooks, and contributing guide (see `docs/README.md`).
- `AGENTES.md`: repository-specific agent guidance, including the explicit Azure-stack override against generic engineering defaults.
- `AGENTS.md`: the project's own root-level agent instructions (stack, non-negotiables, API policy, git conventions), replacing reliance on a generic template outside the repository.
- `documentation-system-builder` skill registered in `skills-lock.json` (from `ljaviertovar/documentation-system-builder`).
- `ADR-0004`: HeroUI v3 (`@heroui/react` + `@heroui/styles`), theme "Rabbit," confirmed as the project's exclusive UI component library — no custom/bespoke components. `heroui-react` skill registered in `skills-lock.json` (from `heroui-inc/heroui`) as the authoritative source for HeroUI v3 docs.

### Notes

- Application code is still the default `create-vite` React + TypeScript scaffold; no Tanqueo product features are implemented yet (see Milestone M1 — Foundations in the PRD roadmap).
