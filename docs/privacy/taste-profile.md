# Privacy & Data Handling

Privacy preferences, data handling expectations, consent boundaries, and retention for Tanqueo. This document states what the PRD and architecture already decide; it does not assert legal compliance beyond that evidence.

## Data Collected

| Data | Source | Purpose |
| --- | --- | --- |
| Auth identity (Microsoft or GitHub account id) | SWA built-in auth | Sign-in, household membership |
| Household membership and role | App-created on first sign-in / invite acceptance | Access control, shared data scoping |
| Car details (make, model, year, plate, fuel type, official consumption) | User input + NRCan lookup | Fuel log context, consumption baseline |
| Receipt photo | Camera or gallery upload | Source image for extraction; kept alongside the fill-up |
| Extracted + confirmed fill-up data (station, date, litres, price, total, odometer) | AI extraction + user review | Core product data — dashboard, history, consumption |
| Raw model output, model name, prompt version | Stored per receipt (FR-11) | Debugging extraction accuracy over time, not a distinct privacy surface but retained as long as the receipt is |

## Storage & Residency

- All data is stored in Canada Central (NFR-04).
- Receipt photos live in a private Blob Storage container; access is only via short-lived (≤ 10 min), single-blob-scoped SAS URLs (NFR-02) — never public.
- Database access requires TLS; the firewall allows Azure services only; the connection password is kept in app settings only, never in source (NFR-03).

## Third-Party Processing

- Receipt images are sent to an Azure OpenAI vision model for extraction. The PRD states the model provider does not retain this data (NFR-04). `[Assumed: per current Azure OpenAI data-use terms at the time the PRD was written — this has not been independently verified in this documentation pass. Re-verify against the live Azure OpenAI data processing terms before relying on this claim for any compliance statement.]`

## Consent & Sharing Boundaries

- Household membership is invite-link based; there is no per-record consent or visibility control beyond the household boundary (FR-04: "All members see and edit every car and fill-up in the household").
- There is no granular role model beyond owner/member (PRD Non-Goals: "multi-household admin tooling" is explicitly out of scope for MVP).
- `[Open question: invite by link only, or also by email address? — affects what identifying information (an email) the household owner must supply to invite someone.]`

## Retention

- `[Open question: keep receipt photos forever, or delete after 12 months? — this is explicitly unresolved in the source PRD and has no default in this documentation pass.]`
- No retention policy exists for extraction metadata (raw model JSON, prompt version) independent of the receipt/fill-up lifecycle; deleting a fill-up removes its receipt photo (FR-22), which is the only stated deletion behavior.
- No account-deletion or data-export flow is described anywhere in the PRD. `[Open question: what happens to a household's data if a member leaves or the household is dissolved?]`

## Compliance Posture

No legal or compliance review has been performed for this project. `[Open question: Tanqueo processes personal data (photos, location-adjacent station names, driving patterns) of Canadian residents; determine whether PIPEDA or another applicable framework requires action before this app is used beyond the founding household. This is not assessed anywhere in the source PRD.]` This document makes no compliance claim and should not be cited as one.
