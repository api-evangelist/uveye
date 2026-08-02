---
name: Share a UVeye inspection with a customer
description: Find the most recent inspection for a vehicle, pull its condition detail, mint a public
  link, and log that the link was shared with the customer.
api: openapi/uveye-public-api-v1-openapi.yml
operations: [listLatestInspections, getInspectionDetails, createInspectionPublicLink, recordInspectionShared]
generated: '2026-08-02'
method: generated
source: openapi/uveye-public-api-v1-openapi.yml + conventions/uveye-conventions.yml + errors/uveye-problem-types.yml
---

# Share a UVeye inspection with a customer

The service-lane flow: a vehicle was scanned, and you want the customer to see the condition report.

## Before you start

- Base URL: `https://api.uveye.dev/v1` (development, as published). Production-US URLs come from the
  Production-US Postman environment.
- Send `uveye-api-key: <key>` on every request in this skill. Generate the key in the Global Keys tab
  of the UVeye Back Office at https://us.backoffice.uveye.app/.
- Every call here is a **POST** with a JSON body. A non-POST returns
  `405 unknown method, please use POST`.

## Steps

1. **Find the inspection** — `listLatestInspections`
   `POST /latest-inspections` with one location filter and one window filter, e.g.
   `{"vin": "<vin>", "amountOfDaysForSearch": 7}` or `{"siteId": "<site id>", "amountOfInspections": 25}`.
   Read `inspections[].inspectionId` from the response. Skip this step if you already hold the
   `inspectionId`, VIN, license plate or barcode.

2. **Check the scan is usable** — `getInspectionDetails`
   `POST /inspection` with exactly one of `inspectionId`, `vin`, `licensePlate` or `uniqueId`.
   Before showing anything to a customer, check `isScanInvalid`. If it is `true`, one or more modules
   failed to upload — inspect `isArtemisModuleInvalid` (tires), `isHeliosModuleInvalid`
   (undercarriage) and `isAtlasModuleInvalid` (exterior) to see which, and consider re-scanning
   rather than sharing a partial report.
   Add `"alertsOnly": true` when you only want alert-severity detections and not warnings.

3. **Mint the link** — `createInspectionPublicLink`
   `POST /public-link` with `{"inspectionId": "<id>"}`. Add `"plainMode": true` for the plain
   rendering. The response `url` is the shareable link and `expirationDate` is when it dies — public
   links expire **30 days** after creation. Store the expiry; do not cache the link past it.

4. **Log the share** — `recordInspectionShared`
   After you actually send the link, `POST /inspection-shared` with
   `{"inspectionId": "<id>", "publicLinkUrl": "<url from step 3>"}`. Success is `200` with an empty
   body. `publicLinkUrl` is optional but include it so the record is complete.

## Rules

- **One identifier per request.** Sending none returns `400 Missing search criteria`.
- **Image URLs expire after 1 hour.** Any `overviewImage`, `coverImage`, `treadImage`, `wallImage` or
  detection image URL in the step-2 response is signed and short-lived. Do not persist them — re-run
  `getInspectionDetails` to get fresh URLs, or fetch the bytes immediately via `getInspectionImage`.
- **There is no idempotency key.** `createInspectionPublicLink` called twice may mint two links.
  Mint once, keep the URL, and re-use it until `expirationDate`.
- **No pagination.** Step 1 is windowed, not paged — widen `amountOfDaysForSearch` or raise
  `amountOfInspections` rather than looking for a cursor.
- On `401 Invalid API key` or `401 API key is disabled`, refresh the key in the Back Office; retrying
  the same key will not recover.
- On `404 Inspection not found`, the inspection expired or the identifier is wrong — go back to
  step 1 and re-discover the `inspectionId`.
