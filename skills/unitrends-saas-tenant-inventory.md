---
name: Inventory SaaS backup tenants and users
description: >-
  Enumerate spanning (SaaS backup) domains/tenants, Microsoft Entra ID
  domains, and their protected users through the Unitrends MSP Public API,
  plus endpoint assets and the latest endpoint agent build.
api: openapi/unitrends-public-api-openapi.json
operations:
- 'GET /v2/spanning/domains'
- 'GET /v2/spanning/domains/{domainId}/users'
- 'GET /v1/entra/domains'
- 'GET /api/epb/v1/assets'
- 'GET /v1/agents/latest'
generated: '2026-07-21'
method: generated
---

# Inventory SaaS backup tenants and users

The spec publishes no operationIds; operations are referenced by method + path
(all verified in openapi/unitrends-public-api-openapi.json).

## Auth

Same as every public API call: OAuth 2.0 client credentials at
`POST https://login.backup.net/connect/token`, then
`Authorization: Bearer {access_token}` against `https://public-api.backup.net`.

## Steps

1. `GET /v2/spanning/domains` — list spanning domains/tenants; add `?include[users]=false` to skip embedded users on the first pass. Prefer v2 over `GET /v1/spanning/domains` (older model).
2. `GET /v2/spanning/domains/{domainId}/users` — page through the users of each tenant of interest.
3. `GET /v1/entra/domains` — list Microsoft Entra ID domains/tenants under protection.
4. `GET /api/epb/v1/assets?customer_id={id}` — endpoint backup assets with `licenseType`, `storageUsedBytes`, `status`, `os`, `backupEnabled`, and `lastSuccessfulBackupTimestamp`.
5. `GET /v1/agents/latest` — latest endpoint agent `version`, `downloadLocation`, and SHA256 `checksum` to verify fleet currency.

## Rules

- Page with `page_number`/`page_size` and read totals from the `Paging-*` response headers.
- Everything is tenant-scoped read-only; there is no write surface in the public API.
