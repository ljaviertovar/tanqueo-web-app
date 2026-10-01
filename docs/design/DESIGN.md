# Design

Product experience, interaction, and accessibility decisions supported by the PRD. Visual system specifics (color tokens, typography scale, spacing) are not yet decided — see the open question at the end.

Status as of 2026-09-30: no UI has been built. `[Inferred from codebase: src/App.tsx is the unmodified Vite scaffold; no Tailwind, HeroUI, or design tokens are installed in package.json yet.]`

**UI component library (confirmed 2026-09-30):** [HeroUI v3](https://heroui.com) (`@heroui/react` + `@heroui/styles`, Tailwind CSS v4 + React Aria, no provider, compound components like `Card.Header`), theme "Rabbit," is the project's exclusive component library — see [`ADR-0004`](../adr/ADR-0004-heroui-as-exclusive-component-library.md). No custom or from-scratch components are built; every screen is composed from HeroUI components. A bespoke component is justified only when a specific case genuinely cannot be solved by any HeroUI component. This supersedes the earlier assumed Tailwind + shadcn/ui default — shadcn/ui does not apply to this project. "Rabbit" is a custom theme (built via HeroUI's Theme Builder), not a shipped preset — see the open question below.

## Product Experience Principles

- **Speed over completeness at capture time.** The core loop (photo → review → save) targets under 30 seconds (PRD Goals). Every review-screen decision should protect that budget: prefill aggressively, only ask for what the receipt can't have (odometer, full-tank flag).
- **Never block on AI.** A failed or low-confidence extraction always degrades to manual entry rather than stopping the flow (FR-12, NFR-08).
- **Confirm, don't silently correct.** Likely duplicates and high consumption are surfaced as an explicit choice to the user, not auto-resolved (FR-16, FR-18).

## Locale & Units

- UI language: English.
- Currency: CAD.
- Fuel: litres; price displayed in ¢/L, matching Ontario pump and receipt conventions.
- Distance: km; consumption: L/100 km.

No localization (additional languages or unit systems) is planned for MVP.

## Responsive Layout

- **Phone (≥ 360 px, primary target).** Header with tabs for navigation; single-column flows for capture, review, and history.
- **Desktop (≥ 1024 px).** Multi-column dashboard plus a recent-fill-ups table (FR-23).

No breakpoint or layout behavior is specified for the 360–1024 px range beyond "responsive" — `[Open question: define the tablet/mid-width behavior, or confirm phone layout stretches until the 1024 px desktop breakpoint.]`

## Accessibility

Target: WCAG 2.2 AA (NFR-07):

- Minimum 4.5:1 text contrast.
- 44 px minimum touch targets.
- All inputs labelled.
- Fully keyboard navigable.
- Lighthouse accessibility score ≥ 95 on phone (PRD success metric).

No accessibility audit has been run yet, since no UI exists.

## Core Screens & States

Derived from the PRD's functional requirements and user flow — not yet designed visually:

| Screen / state | Key behavior |
| --- | --- |
| Add car | Make/model/year/plate form; shows NRCan match for confirmation or allows manual entry (FR-05, FR-06) |
| Capture | Camera or gallery picker; client-side resize to max 2000 px before upload (FR-08) |
| Review | Prefilled fields; confidence < 0.8 highlighted; required odometer input; full-tank toggle; litres×price mismatch warning (FR-13–15) |
| Duplicate sheet | Shows the conflicting existing fill-up and who logged it; default action is "don't save" (FR-16) |
| Saved summary | Shows computed consumption for full-tank fill-ups; surfaces the high-consumption alert with common causes when triggered (FR-17, FR-18) |
| Dashboard | Per-car: monthly spend, consumption (latest vs. official), avg ¢/L, cost/km, price trend (last 8 fill-ups), station spend (90 days, cheapest flagged), spend per month (last 4) (FR-20) |
| History | Month-grouped list; all vs. full-tank-only filter; Full/Partial badge; tap to edit (FR-21, FR-22) |

## PWA

Installable via manifest + service worker; app shell loads offline, data requires a connection (FR-24). Not yet implemented — planned for Milestone M6 (see PRD Roadmap). No app icon set, splash screen, or offline UX has been designed yet.

## Open Questions

- `[Open question: the exact "Rabbit" theme token values (colors, typography) are not yet confirmed against HeroUI's own theming docs — see ADR-0004. Resolve before implementing the first screen.]`
- `[Open question: confirm whether the confidence-highlight and duplicate-sheet interactions need motion/animation specs — HeroUI v3 animations are CSS-based (no framer-motion dependency) — or plain static states are sufficient for MVP.]`
