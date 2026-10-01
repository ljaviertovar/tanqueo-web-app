# Deployment Runbook

Status: **planned, not yet implemented.** No deployment exists today — the repository is pre-Milestone-M1 (Foundations). This runbook documents the target flow implied by the confirmed architecture (`docs/architecture/architecture.md`) so it's ready to execute and refine once M1 starts, not a procedure that has been run and verified.

## Prerequisites

- An Azure subscription with access to create: a Static Web App (Free tier), an Azure Database for PostgreSQL Flexible Server (Burstable B1ms), a Blob Storage account, and an Azure OpenAI resource, all in Canada Central.
- A GitHub repository connected to the Static Web App for CI/CD (Static Web Apps deploys frontend + managed Functions together from GitHub Actions). `[Open question: no GitHub Actions workflow exists yet in this repository.]`
- Database connection string and Azure OpenAI key stored only in the Static Web App's application settings — never in source or `.env.example` (NFR-03, and the ambient `AGENTS.md` secrets policy).
- NRCan Fuel Consumption Ratings dataset available for the one-time `vehicle_ratings` import (FR-06).

## Deployment Steps

`[Open question: this section is a placeholder until Milestone M1 actually stands up the resources above. Fill in with the real, tested steps at that point — do not treat the outline below as verified.]`

1. Provision Azure resources (Static Web App, PostgreSQL Flexible Server, Blob Storage container, Azure OpenAI) in Canada Central.
2. Apply database migrations for the 8-table schema (`docs/architecture/architecture.md` → Core Modules → Data model).
3. Run the one-time NRCan `vehicle_ratings` import.
4. Configure application settings: DB connection string, Blob Storage connection/SAS signing key, Azure OpenAI endpoint and key, budget alert threshold ($5 USD, NFR-06).
5. Connect the GitHub repository to the Static Web App; push to the deploy branch to trigger the build.
6. Verify the deployed app can sign in via Microsoft or GitHub (FR-01) and that the first sign-in creates a household (FR-02).

## Rollback Procedure

`[Open question: no rollback procedure has been defined or tested. At minimum, Static Web Apps supports redeploying a previous GitHub Actions build; database migrations need an explicit down-migration or backup-restore plan before this section can be considered complete.]`

## Verification

- Sign-in works for both configured identity providers.
- A new household is created on first sign-in with the signing-in user as owner.
- An invite link can be created and accepted (FR-03).
- A receipt can be uploaded, extracted, reviewed, and saved end-to-end.
- Dashboard and history endpoints return data scoped to the correct household only (cross-household isolation check — see ADR-0003).
- Application Insights is receiving API error and extraction latency telemetry (NFR-09).
