# Incident Response Runbook

Status: **planned, not yet implemented.** No monitoring, alerting, or on-call process exists today — the repository is pre-Milestone-M1. This runbook documents the target monitoring shape implied by the confirmed architecture and NFRs, to be filled in with real procedures once M1/M6 (Application Insights, backups) ship.

## Monitoring & Detection

- **Application Insights** for API errors and extraction latency (NFR-09) — not yet instrumented.
- **Extraction accuracy** measured by comparing stored AI output against the user's final confirmed values (NFR-09) — not yet built.
- **Budget alert** at $5 USD/month (NFR-06) — not yet configured.
- `[Open question: no alerting destination (email, Slack, etc.) has been decided for either the budget alert or Application Insights alerts.]`

## Common Incident Scenarios

Anticipated from the PRD's risks table, not yet observed in production (there is no production yet):

| Scenario | Signal | First response |
| --- | --- | --- |
| Extraction provider outage or latency spike | Extraction timeouts (> 20 s) climb in Application Insights | Confirm fallback-to-manual-entry path still works (FR-12, NFR-08) — this should degrade gracefully, not fail the app |
| Budget alert fires | Azure budget notification at $5 USD | `[Open question: no automated action is defined — decide whether this pauses extraction, just notifies the owner, or is purely informational]` |
| Cross-household data leak suspected | A user reports seeing another household's data | Treat as highest severity — see ADR-0003 (tenancy is enforced in application code, not DB RLS, so this class of bug has no second line of defense) |
| Database unreachable | API 5xx spike, connection errors in Application Insights | Check firewall rules (Azure-services-only, NFR-03) and TLS configuration first |

## Troubleshooting Guide

`[Open question: no diagnosis/resolution steps have been written yet — this needs real operational experience or, at minimum, a dry run once M1 is deployed.]`

## Escalation Policy

`[Open question: single-maintainer project (per docs/architecture/project-structure.md) — there is no on-call rotation or escalation chain to define yet. Revisit if collaborators join.]`

## Post-Incident Actions

- Record the incident and resolution here or in a linked entry once a real incident occurs.
- If the incident reveals a gap addressed by an existing open question in this documentation system (e.g., the ADR-0003 tenancy gap, or the budget-alert response above), resolve that open question as part of the follow-up rather than only patching the immediate symptom.
