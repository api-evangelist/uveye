---
name: Pull appraisal and reconditioning cost from a UVeye inspection
description: Turn a scanned vehicle into an appraisal — offer price, total reconditioning cost, and
  damages grouped by car part with supporting images.
api: openapi/uveye-public-api-v1-openapi.yml
operations: [listLatestInspections, getQuoteByInspection, getInspectionImage]
generated: '2026-08-02'
method: generated
source: openapi/uveye-public-api-v1-openapi.yml + conventions/uveye-conventions.yml + errors/uveye-problem-types.yml
---

# Pull appraisal and reconditioning cost from a UVeye inspection

The trade-in / remarketing flow: a vehicle was scanned and an appraisal exists on uvcamp; you want
the numbers and the damage breakdown.

## Before you start

- Send `uveye-api-key: <key>` on every request. Base URL `https://api.uveye.dev/v1`.
- **The inspected vehicle must have a related appraisal ready on uvcamp.** Without it,
  `getQuoteByInspection` returns `404` even though the inspection exists.

## Steps

1. **Resolve the inspection** — `listLatestInspections` (skip if you already have an identifier)
   `POST /latest-inspections` with `{"vin": "<vin>", "amountOfDaysForSearch": 30}`. Take
   `inspections[].inspectionId`.

2. **Get the quote** — `getQuoteByInspection`
   `POST /quote` with exactly one of `inspectionId`, `licensePlate`, `vin` or `uniqueId`, plus
   `"includeImages": true` when you want damage imagery.
   The response carries basic vehicle information, the public quote link, the offer price and the
   total reconditioning cost when available, and `defects[]`.

3. **Walk the damage graph.** The nesting is three deep and easy to get wrong:
   - each `defects[]` entry is **one car part**;
   - that entry's `damages[]` array holds the **estimated repairs** for that part (estimator
     description, cost to repair or replace);
   - each repair carries its **own** `damages[]` array of supporting images, each with an image URL
     and the damage-location rectangle for the detection.

4. **Fetch imagery if you need the bytes** — `getInspectionImage`
   `GET /image?key=<key>`. You do not construct these URLs: the URLs returned in step 2 already carry
   `?key=`. Use them as-is. The response is raw `image/jpeg` (or `image/png`), not JSON.

## Rules

- **Image URLs are valid for 1 hour.** If you are building a report asynchronously, download the
  bytes during the same session or re-run step 2 for fresh URLs.
- **One identifier per request.** None supplied returns `400 Missing search criteria`.
- A `404` from step 2 means no inspection **or no active quote** for that identifier — check uvcamp
  before assuming the identifier is wrong.
- `401 Invalid API key` / `401 API key is disabled`: refresh the key in the UVeye Back Office.
- Prices and reconditioning totals are returned "when available" — treat both as optional and do not
  assume a quote is complete.
