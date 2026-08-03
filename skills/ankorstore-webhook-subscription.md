---
name: Subscribe to order webhooks
description: Register an Ankorstore webhook subscription and verify delivery signatures.
api: openapi/ankorstore-main-openapi-original.yml
operations: [list-applications, get-available-webhook-events, add-application-webhook-subscription]
---

# Subscribe to Ankorstore webhooks

Build event-driven integrations instead of polling.

## Auth
Use an OAuth2 client-credentials Bearer token with `Accept: application/vnd.api+json`.

## Steps
1. **Find your application** — `list-applications` (`GET /api/v1/applications`). Webhook subscriptions are tied to an OAuth application.
2. **Discover events** — `get-available-webhook-events` to list subscribable event types (e.g. `order.brand_created`, `order.brand_accepted`, `order.shipped`, `order.brand_paid`, `external_order.*`).
3. **Subscribe** — `add-application-webhook-subscription` (`POST /api/v1/webhook-subscriptions`) with your public HTTPS endpoint and the events you want.

## Verify deliveries
- Each delivery carries a `signature` header = `HMAC-SHA256(rawPayload, subscriptionSecret)` in hex. The subscription secret is per-subscription and is NOT your API client secret.
- Recompute the HMAC over the raw body and compare with a constant-time function (`crypto.timingSafeEqual` / `hmac.compare_digest`). Never use `==`.
- The payload is the JSON:API resource object plus `meta.event` (`id`, `applicationId`, `type`, `timestamp`).

## Rules
- Return a `2xx` quickly; non-2xx triggers up to 5 retries with exponential backoff. Make your handler idempotent (dedupe on `meta.event.id`).
- Endpoint must use HTTPS with a valid TLS certificate.
