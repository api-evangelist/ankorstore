---
name: Process and ship an order
description: Accept (or reject) a new Ankorstore order and drive it through the shipping flow.
api: openapi/ankorstore-main-openapi-original.yml
operations: [list-master-orders, get-master-order, brand-accept-order, brand-reject-order, list-order-shipping-quotes, confirm-order-shipping-quote, ship-internal-order-schedule-pickup]
---

# Process and ship an Ankorstore order

## Auth
Obtain an OAuth2 client-credentials Bearer token from `POST /oauth/token` (scope=*) and send it with `Accept: application/vnd.api+json`.

## Steps
1. **Find new orders** — `list-master-orders` (`GET /api/v1/master-orders?include=internalOrder,externalOrder`), filtered e.g. `filter[status]=ankor_confirmed`. Prefer subscribing to the `order.brand_created` webhook over polling.
2. **Read the order** — `get-master-order` (`GET /api/v1/master-orders/{masterOrder}?include=internalOrder`) for lines, amounts and shipping address.
3. **Accept** — `brand-accept-order` to accept the internal order, OR
4. **Reject** — `brand-reject-order` with a `rejectType` from the documented reasons (e.g. `PRODUCT_OUT_OF_STOCK`, `RETAILER_NOT_GOOD_FIT_FOR_BRAND`, or `OTHER` + free-text `rejectReason`). See errors/ankorstore-error-codes.yml.
5. **Get quotes** — `list-order-shipping-quotes` (`POST /api/v1/orders/{order}/shipping-quotes`) with parcel dimensions/weight.
6. **Confirm a quote** — `confirm-order-shipping-quote` (`POST /api/v1/shipping-quotes/{quote}/confirm`); generates a label when using Ankorstore Label.
7. **Schedule pickup** — `ship-internal-order-schedule-pickup` (`POST /api/v1/orders/{order}/ship/schedule-pickup`) for Ankorstore Label shipments.

## Rules
- A `409 Conflict` (e.g. `ORDER_ACTION_REJECT_FAILED`) means the order is not in a state the action allows — re-fetch and re-evaluate.
- Supply `data.id` (a client UUID) on creates for idempotent retries.
- Confirm completion via webhooks (`order.brand_accepted`, `order.shipped`, `order.shipment_received`).
