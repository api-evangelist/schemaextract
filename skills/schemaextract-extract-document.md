---
name: extract-document-to-json
description: >-
  Extract structured JSON from a PDF or image with SchemaExtract, using a built-in preset
  (invoice, packing_list, bank_statement, bol) or a caller-supplied JSON schema.
api: SchemaExtract API
generated: '2026-09-20'
method: generated
source: openapi/schemaextract-openapi.yml
operations:
  - extractDocument
---

# Extract a document to JSON

Turn a PDF or image into typed JSON using the SchemaExtract `extractDocument` operation
(`POST /v1/extract`). You either name a built-in **preset** or paste your own **JSON schema**;
the API returns only the fields you asked for. Missing values come back `null`; extra keys are
never invented.

## When to use
- You have an invoice, packing list, bank statement, or bill of lading (or any document) and need
  it as structured data.
- You want a specific output shape — supply a JSON schema instead of a preset.

## Steps

1. **Pick your target shape.** Either set `preset` to one of `invoice | packing_list |
   bank_statement | bol`, or supply a `schema` (a JSON Schema or example object, object root, max
   50 top-level keys, depth 8, 200 total keys, 64 KB).
2. **Send the document.** Two ways:
   - multipart form-data with `file` (PDF/PNG/JPG/WebP, max 10 MB, first 3 pages) plus `preset`
     and/or `schema`;
   - or a JSON body with `file_base64`, `filename`, `mime`, plus `preset`/`schema`.
   ```sh
   curl -sS -X POST https://schemaextract.shop/v1/extract \
     -F "preset=invoice" \
     -F "file=@invoice.pdf"
   ```
   On the free demo (`DEMO_MODE`) no auth header is needed. Otherwise send
   `Authorization: Bearer sk_live_…`.
3. **Read the result.** A success is `{ "ok": true, "data": {…}, "meta": {…} }` where `data` matches
   your preset/schema and `meta` reports `model`, `pages`, `preset`, `mode`, `plan`, and
   `quota_remaining`.
4. **For slow/large runs, go async.** Use `POST /v1/extract/jobs` (or `POST /v1/extract?async=1`
   with `Prefer: respond-async`), then poll the returned job URL — proxies can drop idle POSTs.

## Conventions & guardrails
- **Rate limits:** 60 requests / 10 minutes per client → `429 RATE_LIMITED` (inspect `X-RateLimit-*`).
- **Quota:** Free 20 / Starter 500 / Pro 5000 extracts per month → `402 QUOTA_EXCEEDED`.
- **Errors** use a custom envelope `{ "ok": false, "error": { "code", "message" } }`; codes include
  `INVALID_SCHEMA`, `UNSUPPORTED_FILE`, `FILE_TOO_LARGE`, `MODEL_FAILURE`. See
  `errors/schemaextract-problem-types.yml`.
- **Privacy:** uploaded bytes are processed in memory and discarded after the response; only account
  metadata is stored. Process only documents you are allowed to handle.
