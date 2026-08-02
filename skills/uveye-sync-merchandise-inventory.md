---
name: Sync dealer inventory to UVeye Merchandise
description: Push VINs to UVeye for merchandising imagery, receive the images-ready webhook, and mark
  vehicles sold when they leave the lot.
api: openapi/uveye-public-api-v1-openapi.yml
operations: [submitMerchandiseInventory, markMerchandiseVehiclesSold]
generated: '2026-08-02'
method: generated
source: openapi/uveye-public-api-v1-openapi.yml + asyncapi/uveye-merchandise-webhooks.yml +
  conventions/uveye-conventions.yml + errors/uveye-problem-types.yml
---

# Sync dealer inventory to UVeye Merchandise

The merchandising loop: you send your active inventory, UVeye scans and renders multi-angle imagery,
and pushes it back to your webhook.

## Before you start

- **Merchandise uses a different credential from the rest of the API.** Send
  `Authorization: Bearer <merchandise API key>` — the key issued to you at onboarding. Do **not**
  send `uveye-api-key` here; do not send the merchandise key to the inspection endpoints.
- Your webhook URL is configured by UVeye at onboarding. There is no self-service endpoint to change
  it.

## Steps

1. **Submit inventory** — `submitMerchandiseInventory`
   `POST /merchandise/inventory/vehicles` with `{"vehicles": [...]}`.
   - Required per vehicle: `vin`, `make`, `model`, `year` (integer 1900–2100).
   - Recommended: `color`, `body` — these improve the rendered output.
   - Optional UI metadata only, no effect on images: `stockNumber`, `trim`, `mileage`, `type`
     (`NEW`/`USED`), `sellingPrice`, `interiorColor`, `transmission`, `drivetrain`, `engine`,
     `fuelType`, `msrp`, `certified`, `photoCount`, `stockInDate` (`YYYY-MM-DD`).
   - **Batch limit is 100 vehicles.** More returns `413`. Chunk larger inventories.
   - The response is `202 Accepted` with `{received, requestId, count}`. **Keep `requestId`** — it is
     the only handle support has on the batch.

2. **Run it on a cadence.** Resubmit your **full active inventory** periodically — roughly every 4
   hours is the documented pattern. Unchanged vehicles are no-ops, deduplicated by content hash, so
   resubmission is safe and cheap. This is the API's idempotency model; there is no
   `Idempotency-Key` header.

3. **Receive the images-ready webhook.** When a VIN's images are ready UVeye POSTs to your URL:
   - `Authorization: Bearer <customer-bearer-token>` — the token UVeye was given at onboarding.
     Match it to authenticate that the caller really is UVeye.
   - `X-UVeye-Signature` — an HS256 JWT binding the body via `body_sha256`. Verify it for tamper
     detection; it also carries a `delivery_id` you can use as an idempotency key.
   - Default body is minimal: `{vin, publishedAt, images[]}`. The detailed format (opt-in at
     onboarding) adds `publishStatus`, `coverImage`, `imageCount`, per-image `{url, category}` and
     `modules`.
   - **Acknowledge with 2xx.** A non-2xx triggers retry — up to 5 retries / 6 attempts total, all
     sharing the same `delivery_id`. Deduplicate on `delivery_id`.

4. **Mark vehicles sold** — `markMerchandiseVehiclesSold`
   `POST /merchandise/inventory/sold` with `{"vins": ["...", "..."]}`, same 100-item batch limit,
   same `202 Accepted` shape.
   **You must call this explicitly.** v1 does **not** infer that a vehicle sold from its absence in a
   resubmitted inventory. To bring a sold VIN back, just resubmit it through step 1.

## Rules

- **Nothing is validated per VIN in the response.** Batches are processed asynchronously; a malformed
  or missing-field VIN is silently excluded and simply never produces a webhook. If a VIN never
  arrives, check its required fields first — there will be no error telling you.
- `400`: malformed body or a missing required field (step 1); missing/empty `vins` or a non-string
  entry (step 4).
- `401`: the merchandise key is missing, invalid, disabled, or not authorized for merchandise — this
  is a distinct authorization from your inspection API key.
- `413`: more than 100 vehicles in one request.
- Reconcile by counting: compare the `count` echoed in each `202` against the webhooks you receive,
  and hold the `requestId` for anything that does not reconcile.
