# ADR 0002 — Extract receipt data via a vision model behind a swappable interface

## Status

Accepted (recorded in the source PRD, Sep 19, 2026).

## Context

Gas receipts vary widely by station chain, printer condition, and layout; thermal receipts fade and crumple. Tanqueo needs to extract station, date, litres, price per litre, and total from a photo in a few seconds, with enough confidence signal to know when to trust the result, while keeping the option open to change extraction providers if accuracy or cost don't hold up.

## Decision

Use an Azure OpenAI vision model with structured outputs (a fixed JSON schema, `additionalProperties: false`) to extract fields plus a per-field confidence score. All extraction code depends on one interface:

```ts
interface ReceiptExtractor {
  extract(imageUrl: string): Promise<ExtractedReceipt>;
}
```

The first (and currently only) implementation is `AzureOpenAIReceiptExtractor`. Every receipt stores the model's raw JSON output, model name, and prompt version (FR-11), and carries a `status` field (`uploaded → extracting → extracted → confirmed | failed`) so the extraction step can later move from a synchronous HTTP call to an asynchronous queue-triggered Function without changing the data model or the frontend.

## Alternatives Considered

- **Rule-based / per-chain OCR parsing.** Rejected: would require per-station-chain rules and breaks on receipt layout changes; a vision model with structured outputs generalizes across layouts without that maintenance burden.
- **Azure Document Intelligence** (a receipt/OCR-specific service). Not chosen as the first implementation, but explicitly kept open as a second implementation (`AzureDocumentIntelligenceReceiptExtractor`) to compare accuracy against the vision-model approach, made possible by the shared interface.

## Consequences

- Positive: swapping or adding an extraction provider touches only the interface implementation, not calling code (FR-12's fallback-to-manual-entry path, the review form, and dashboard aggregation are all extractor-agnostic).
- Positive: storing raw output, model name and prompt version per receipt (FR-11) gives a concrete way to measure accuracy drift or compare providers later (NFR-09).
- Negative: dependent on a single external provider's availability and latency for the primary path; mitigated by NFR-08 ("a failed extraction never blocks saving") and the 20 s timeout in FR-12.
- Negative: no automated accuracy regression test exists yet — accuracy is measured only via the stored AI-vs-final-value comparison (NFR-09), which is an observability measure, not a pre-deploy gate. `[Open question: should extraction accuracy have a pre-deploy test gate against a fixed receipt sample set?]`
