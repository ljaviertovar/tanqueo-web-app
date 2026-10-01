# Tooling & Delegation (Skills + MCP)

Tool ownership, delegation boundaries, permitted operations, approvals, and escalation rules for agents working in this repository. This is the canonical document `AGENTS.md` points to for this topic.

## Skill Registry

Skills are installed via the `skills` CLI (`npx skills install` to bootstrap, `npx skills add <source>` to add one) and tracked in `skills-lock.json` at the repo root, with the actual skill content synced into `.agents/skills/` (and symlinked into per-agent directories such as `.claude/skills/`).

| Skill | Source | Purpose |
| --- | --- | --- |
| `vercel-react-best-practices` | `vercel-labs/agent-skills` (GitHub) | React/Next.js performance optimization guidelines; reference when writing or reviewing React components |
| `documentation-system-builder` | `ljaviertovar/documentation-system-builder` (GitHub) | Build, audit, migrate and maintain this documentation system |
| `heroui-react` | `heroui-inc/heroui` (GitHub) | Authoritative HeroUI v3 docs/component fetcher — use for every HeroUI component or theming task instead of relying on memory (v2 patterns do not apply); see [`ADR-0004`](adr/ADR-0004-heroui-as-exclusive-component-library.md) |

Re-run `npx skills install` whenever new skills are added, per `AGENTS.md`'s Setup section. When adding a skill from a local path, verify the resulting `skills-lock.json` entry: a `sourceType: "local"` entry stores an absolute machine path and will not restore on another machine or in CI — prefer registering skills from a Git/GitHub source so `experimental_install` can restore them anywhere.

## Delegation Boundaries

**Agents may do without asking:**

- Read-only inspection: reading files, searching, running `git status`/`git log`/`git diff`.
- Local, reversible edits: creating or editing files, running the dev server, running lint/typecheck/test commands.
- Generating or updating documentation under `docs/` when explicitly requested.

**Agents must confirm before doing:**

- Destructive or hard-to-reverse git operations: `git reset --hard`, force-push, branch deletion, discarding uncommitted changes.
- Any database migration that could remove or alter existing records — this repository has no database yet, but the rule carries forward to whichever Postgres instance (Azure Flexible Server, per `docs/architecture/architecture.md`) is provisioned at Milestone M1.
- Secret or credential handling of any kind: connection strings, SAS-generating keys, Azure OpenAI keys. None of these belong in source, `.env.example` (names/empty values only), or chat output.
- Pushing to a shared branch, opening/closing/commenting on PRs or issues, or any action visible to other collaborators.
- Deploying to Azure Static Web Apps or any other shared/production environment.

**Default to deny when unsure** — this mirrors the ambient `AGENTS.md`'s security posture and applies to tooling decisions, not just authorization code.

## Escalation

If a task requires an action outside the "may do without asking" list, stop and ask the user rather than proceeding — do not route around a decline by finding an alternate destructive path to the same result.

## Open Items

- `[Open question: no CI/CD pipeline exists yet; once one is added (likely GitHub Actions deploying to Azure Static Web Apps per the architecture doc), this document should gain a section on what agents may trigger versus what requires human approval in that pipeline.]`
