# ADR 0004 — Use HeroUI as the exclusive UI component library, theme "Rabbit"

## Status

Accepted (product owner decision, 2026-09-30).

## Context

`docs/design/DESIGN.md` and `AGENTS.md` previously carried only an assumed default (Tailwind CSS + shadcn/ui, inherited from a generic engineering template) — no Tanqueo-specific UI kit had been chosen, and nothing was installed in `package.json`. The product owner has now made an explicit choice: [HeroUI](https://heroui.com) v3 as the component library, with a theme named "Rabbit."

HeroUI v3 is a React component library built on **Tailwind CSS v4** and **React Aria Components**. It is a significant rewrite from the earlier v2/NextUI lineage (which some general-purpose documentation and search results still describe) — the two are not interchangeable:

| Aspect | v2 (does not apply here) | v3 (this project) |
| --- | --- | --- |
| Provider | `<HeroUIProvider>` required | **No provider needed** |
| Component API | Flat props (`<Card title="x">`) | Compound components (`<Card><Card.Header>…`) |
| Styling | Tailwind v3 + `@heroui/theme` | Tailwind v4 + `@heroui/styles` |
| Animation | `framer-motion` dependency | CSS-based, no extra dependency |
| Packages | `@heroui/react` (+ `@heroui/system`, `@heroui/theme`) | `@heroui/react` + `@heroui/styles` |

[Verified via the project's `heroui-react` skill and HeroUI's own v3 quick-start doc, fetched 2026-09-30: `https://heroui.com/en/docs/react/getting-started/quick-start`.] Requirements: **React 19+** (already satisfied — `package.json` has React 19.2.8) and Tailwind CSS v4.

`[Open question: "Rabbit" could not be found as a named, HeroUI-shipped preset theme in official v3 documentation. HeroUI v3 ships one default theme plus a visual "Theme Builder" (`heroui.com/themes`) for creating and exporting a custom theme as a CSS block keyed by `[data-theme="<name>"]` — the docs' own worked example is a custom theme literally named "ocean," built this way, not a preset. Treat "Rabbit" as a custom theme name the product owner intends to define (via the Theme Builder or hand-written CSS variables) rather than an existing preset to merely select. Capture the actual `[data-theme="rabbit"]` CSS variable values here once built/exported — this ADR does not invent them.]`

## Decision

1. HeroUI v3 (`@heroui/react` + `@heroui/styles`) is the project's one and only UI component library, themed "Rabbit" (a custom theme, per the open question above, to be defined via HeroUI's Theme Builder and applied as `<html data-theme="rabbit">`).
2. **No custom or from-scratch components.** Every UI need is built by composing HeroUI's compound components (e.g. `Card.Header`, `Card.Title`). A bespoke/custom component may be created only when a specific requirement genuinely cannot be met by any HeroUI component, alone or composed — and that justification should be stated where the component is introduced (PR description or a code comment), not assumed.
3. This supersedes the earlier assumed default of Tailwind CSS + shadcn/ui. Tailwind CSS v4 remains (HeroUI is built on it), but shadcn/ui (a copy-into-repo, customize-freely primitive set) does not apply — it is a different model from HeroUI's installed-library, compose-don't-fork approach.
4. Use `onPress` (React Aria event) rather than `onClick` on HeroUI interactive components, per HeroUI's own accessibility guidance.

## Alternatives Considered

- **shadcn/ui** (the previous assumed default). Rejected by explicit product decision: shadcn/ui's copy-into-the-repo model invites exactly the kind of bespoke/custom component drift this decision is meant to prevent.
- **HeroUI v2** (the more mature, longer-established NextUI-lineage version). Not chosen — the product owner specified HeroUI without a version, and v3 is the version now presented as current at `heroui.com`. `[Open question: if v3's component surface proves too thin for a needed screen, re-evaluate v2 explicitly rather than defaulting to a custom component.]`
- **Continuing with no chosen library** (plain Tailwind markup per screen, as `docs/design/DESIGN.md`'s screen descriptions implicitly assumed). Rejected — this is what the "no custom components" rule now explicitly forecloses.

## Consequences

- Positive: one consistent, accessible-by-default (React Aria) component set across the app; less bespoke UI code to maintain; React 19 requirement is already satisfied by the current scaffold.
- Positive: `docs/design/DESIGN.md`'s prior open question about "no visual design system exists" is resolved at the library level — HeroUI + the "Rabbit" theme is now the system, once its token values are defined.
- Negative / risk: HeroUI v3 is a newer rewrite than v2; its component surface may be thinner. If a needed component doesn't exist yet, check the component list (via the `heroui-react` skill's `list_components.mjs`) and consider v2 explicitly before reaching for a custom component — don't silently build bespoke UI to route around a v3 gap.
- Negative: none of `@heroui/react`, `@heroui/styles`, or Tailwind CSS v4 are in `package.json` yet — this is a decision recorded ahead of implementation, consistent with the project's pre-M1 status.
- Negative: the exact "Rabbit" theme tokens are undefined until built in the Theme Builder — no screen should be implemented against guessed color values.

## Implementation Notes (for when M1/UI work starts)

- Install: `npm i @heroui/styles @heroui/react` (or the project's package manager, `pnpm`).
- In `src/index.css`: `@import "tailwindcss";` then `@import "@heroui/styles";` — order matters, Tailwind first.
- No `HeroUIProvider` wrapper in `src/main.tsx`.
- Apply the theme via `<html data-theme="rabbit">` (or the project's chosen light/dark class strategy) once the "Rabbit" theme's CSS variables are defined.
- Before implementing any component, fetch its docs via the `heroui-react` skill rather than relying on memorized v2 patterns.

## References

- HeroUI v3 quick-start (requirements, install, no-provider confirmation): `https://heroui.com/en/docs/react/getting-started/quick-start`, fetched 2026-09-30.
- HeroUI v3 theming (CSS-variable/oklch system, Theme Builder, custom-theme worked example): `https://heroui.com/en/docs/react/getting-started/theming`, fetched 2026-09-30.
- v2-vs-v3 API differences: this project's `heroui-react` skill (`.agents/skills/heroui-react/SKILL.md`).
