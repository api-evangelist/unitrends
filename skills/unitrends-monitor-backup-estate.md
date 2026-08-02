---
name: Monitor a Unitrends backup estate
description: >-
  Authenticate to the UniView / Unitrends MSP Portal Public API and walk the
  tenant's estate: customers, appliances, protected assets, recent backups,
  and BackupIQ alerts.
api: openapi/unitrends-public-api-openapi.json
operations:
- 'GET /v1/customers'
- 'GET /v1/appliances'
- 'GET /v1/assets'
- 'GET /v1/backups'
- 'GET /v1/backupiq/alerts'
generated: '2026-07-21'
method: generated
---

# Monitor a Unitrends backup estate

The spec publishes no operationIds; operations are referenced by method + path
(all verified in openapi/unitrends-public-api-openapi.json).

## Auth

1. Obtain a Client ID / Client Secret in the UniView Portal (API Access view > + New). The secret is shown once.
2. `POST https://login.backup.net/connect/token` with `grant_type=client_credentials`, authenticating with HTTP Basic `client_id:client_secret`.
3. Send `Authorization: Bearer {access_token}` on every request to `https://public-api.backup.net`. Visibility is restricted to the tenant that issued the credentials.

## Steps

1. `GET /v1/customers` — page through customers (`page_number`, `page_size` default 50; totals come back in the `Paging-Total-Records` / `Paging-Total-Pages` response headers).
2. `GET /v1/appliances?customer_id={id}` — check `is_online`, `total_mb_size` vs `total_mb_free`, and `version` per appliance.
3. `GET /v1/assets?customer_id={id}&include[backups]=last&include[links]=all` — protected assets with their embedded `last_backup` and portal links.
4. `GET /v1/backups?customer_id={id}&status=Failed&start_time_from={iso}` — recent failed backups; `status` is one of Successful, Warning, Failed, InProgress, Unknown.
5. `GET /v1/backupiq/alerts?type=alert&severity=critical` — surface critical alerts; `type` is required (alert, job, conditional, helix). Conditional alerts include an `interrogation_message` with the diagnosed root cause.

## Rules

- All operations are read-only GETs; there is no write surface, so retries are safe.
- Order results with `order_by` + `order_direction` ("asc"/"desc"); enums per resource are in the spec.
- The spec documents no error envelope or rate limits (see conventions/unitrends-conventions.yml); treat non-200s defensively.
