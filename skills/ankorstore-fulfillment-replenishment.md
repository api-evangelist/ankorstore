---
name: Replenish and track fulfillment
description: Declare incoming stock to an Ankorstore Fulfillment Center and track fulfillments.
api: openapi/ankorstore-main-openapi-original.yml
operations: [fulfillment-create-replenishment, fulfillment-patch-replenishment, fulfillment-list-fulfillable, fulfillment-get-order]
---

# Replenish and track Ankorstore fulfillment

## Auth
Use an OAuth2 client-credentials Bearer token with `Accept: application/vnd.api+json`.

## Steps
1. **Declare incoming stock** — `fulfillment-create-replenishment` (`POST /api/v1/fulfillment/replenishments`) to create a replenishment with its items.
2. **Confirm / update** — `fulfillment-patch-replenishment` (`PATCH /api/v1/fulfillment/replenishments/{replenishment}`) to move the status to `confirmed` when the shipment is ready.
3. **Check availability** — `fulfillment-list-fulfillable` (`GET /api/v1/fulfillment/fulfillable`) to see fulfillable stock. For multi-location, lot-level detail use the ASTRAL `list-state` operation.
4. **Track fulfillments** — `fulfillment-get-order` (`GET /api/v1/fulfillment/orders/{order}?include=statusUpdates`) to follow a fulfillment order's progress.

## Rules
- Supply `data.id` (a client UUID) on POST creates so retries after a network error return the existing resource rather than duplicating it.
- Content-Type must be `application/vnd.api+json` on write bodies.
- Handle `422` via `source.pointer`, `429` via `Retry-After`, `500` via exponential backoff.
