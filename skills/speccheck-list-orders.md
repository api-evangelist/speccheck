---
name: List and page through SpecCheck orders
description: Authenticate and retrieve a lab's orders with date filtering and cursor pagination.
api: openapi/speccheck-openapi.yml
operations: [createAccessToken, listLabs, listOrders]
---

# List and page through orders

## Steps
1. `createAccessToken` (`POST /v1/oauth/token`) with `client_id`/`client_secret`; keep the bearer `token`.
2. `listLabs` (`GET /v1/labs`) with `Authorization: Bearer <token>` and `User-Email` → choose a `lab.id` + `account_number`.
3. `listOrders` (`GET /v1/orders`) with required `lab` and `account_number` query params. Orders come back newest-first.

## Pagination & filtering
- Page size: `limit` (1–100, default 10). Response includes `has_more`.
- Forward page: pass `starting_after=<last order id>`. Back page: `ending_before=<first order id>`.
- Filter by creation time with `created[gte]`, `created[gt]`, `created[lte]`, `created[lt]` (integer Unix seconds).
- Loop while `has_more` is true, advancing `starting_after` to the last `data[].id`.

## Rules
- Each order object exposes `id`, `tray_number`, `invoice_number`, `lab_status`, `created`, `updated` (Unix seconds).
- Do not send `Idempotency-Key` on GET requests. Handle failures via the `error` object (see errors/speccheck-problem-types.yml).
