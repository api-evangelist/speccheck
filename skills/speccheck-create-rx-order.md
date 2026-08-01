---
name: Create a prescription (rx) order with SpecCheck
description: Authenticate, discover a lab and its catalog, then create and submit an rx eyewear order.
api: openapi/speccheck-openapi.yml
operations: [createAccessToken, listLabs, retrieveOrderSettings, listLensStyles, listLensMaterials, listLensAddons, createOrder]
---

# Create a prescription (rx) order

Use this to place a prescription eyewear order with an optical lab through SpecCheck.

## Auth
1. Call `createAccessToken` (`POST /v1/oauth/token`) with your `client_id` and `client_secret`. Use staging credentials against `https://api-staging.speccheckrx.com` while testing. Store the returned `token` (valid 24h).
2. On every later request send `Authorization: Bearer <token>` and a `User-Email` header that exactly matches the acting user's SpecCheck Dashboard login email.

## Discover the lab and catalog
3. `listLabs` (`GET /v1/labs`) → pick a `lab.id` and its `account_number`.
4. `retrieveOrderSettings` (`GET /v1/labs/{id}/order_settings`) → read `order_types` (rx is always available), `mounting_options`, `frame_materials`, `frame_handling_options`, `services`.
5. Drill the lens catalog for that `(lab, account_number)`: `listLensStyles` (pass `lens_type` = `single_vision` or `multifocal`) → `listLensMaterials` (pass `style`) → `listLensAddons` (pass `style` + `material`) to get valid `coats`, `colors`, and `tints`.

## Create the order
6. `createOrder` (`POST /v1/orders`). Set `order_type: rx`, `lab`, `account_number`, `patient` (first_name/last_name), a `frame` object (handling/manufacturer/model/material/color/eye_size/bridge), a `mounting_option`, a `lens` object referencing catalog IDs from step 5, and an `rx` object with `left_eye` and/or `right_eye` powers.
7. Choose `mode`: `draft` to review in the Dashboard, or `submit` to send to the lab (a `lens` block is required when `order_type` is rx and `mode` is submit).
8. Send an `Idempotency-Key` header (a v4 UUID) so retries never duplicate the order. Never put patient data in the key.

## Rules
- Prescription ranges are validated (sphere -30..30, axis 1..180, `cylinder`↔`axis` and `add`↔`seg_height` are mutually required). At least one of `left_eye`/`right_eye` is required for rx.
- Errors come back as `{ "error": { "type", "message", "code?", "param?" } }`; branch on `error.type` (`invalid_request_error` 400, `authentication_error` 401, `permission_error` 403, `api_error` 500) together with the HTTP status. See errors/speccheck-problem-types.yml.
