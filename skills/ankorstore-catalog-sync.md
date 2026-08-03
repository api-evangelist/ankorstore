---
name: Sync catalog and stock
description: Keep an Ankorstore brand catalog and stock levels in sync with an external ERP/PIM.
api: openapi/ankorstore-main-openapi-original.yml
operations: [list-products, get-product, update-product-variant-stock, update-product-variant-prices, post-catalog-integration-operations]
---

# Sync catalog and stock with Ankorstore

Use the Ankorstore Public API (JSON:API) to keep products, stock and prices aligned with your systems.

## Auth
1. POST `https://www.ankorstore.com/oauth/token` (sandbox: `https://www.public.ankorstore-sandbox.com/oauth/token`) with `grant_type=client_credentials`, your `client_id`/`client_secret`, and `scope=*`. Cache the Bearer token (valid 3600s); refresh ~5 min before expiry. Do NOT re-auth per call — `/oauth/token` allows only 60/hour.
2. Send `Authorization: Bearer {token}` and `Accept: application/vnd.api+json` on every call.

## Steps
1. **List products** — `list-products` (`GET /api/v1/products`). Page with `page[limit]` and follow `links.next` (cursor pagination).
2. **Inspect a product** — `get-product` (`GET /api/v1/products/{product}?include=productVariants`) to read its variants.
3. **Update stock** — `update-product-variant-stock` (`PATCH /api/v1/product-variants/{productVariant}/stock`). PATCH-to-value is idempotent; safe to retry.
4. **Update prices** — `update-product-variant-prices` (`PATCH /api/v1/product-variants/{productVariant}/prices`).
5. **Bulk changes** — for initial import or large create/update/delete batches, use `post-catalog-integration-operations` (`POST /api/v1/catalog/integrations/operations`) for async processing with callbacks.

## Rules
- Content-Type must be `application/vnd.api+json` on PATCH/POST bodies.
- Handle `422` per-field via `source.pointer`; `429` → wait `Retry-After`; `500` → exponential backoff (max 5 attempts).
- Prefer webhooks over polling for downstream state (see the webhook-subscription skill).
