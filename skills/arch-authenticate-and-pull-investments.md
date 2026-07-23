---
name: Authenticate and pull private-market investment data
description: Get a JWT access token with client credentials, then read holdings, cash flows, and the investing entities behind them from the Arch Client API.
api: openapi/arch-client-api-openapi.json
operations:
  - POST /client-api/v0/auth/token
  - GET /client-api/v0/investing-entities
  - GET /client-api/v0/holdings
  - get-single-investing-entity
  - GET /client-api/v0/cash-flows
---

# Authenticate and pull investment data

Use this to read a client's private-market portfolio from Arch.

## Prerequisites
- A Client ID and Client Secret (request from api-support@arch.co).
- Base URL: `https://arch.co/client-api/v0`.

## Steps

1. **Get a token** — `POST /client-api/v0/auth/token` with JSON body `{ "clientId": "...", "clientSecret": "..." }`. The response contains a JWT. Send it on every subsequent call as `authorization: Bearer <token>`. Store and reuse it until it expires — decode the JWT `exp` (Unix seconds) to know when to refresh.
2. **List investing entities** — `GET /client-api/v0/investing-entities` to enumerate the entities that own holdings. Get one with `get-single-investing-entity` (`GET /client-api/v0/investing-entities/{id}`); add `includeWireInstructions=true` when you need wire details.
3. **List holdings** — `GET /client-api/v0/holdings` (paginated). Filter with `investingEntityIds`, `issuingEntityIds`, or `accountIds`, and hydrate related data with `include*` flags (`includeCashFlow`, `includeFiles`, `includeMetrics`, `includeTax`).
4. **Read cash flows** — `GET /client-api/v0/cash-flows` for capital calls / distributions; filter by date with `afterCreatedAt` / `beforeCreatedAt`.

## Rules
- **Pagination**: results come back as "Pages" — use `offset` and `limit` (default 25, max 1000); loop until you have all `contents`.
- **Rate limits**: per-minute; on HTTP 429 back off and honor `RateLimit-Reset`. Watch `RateLimit-Remaining` proactively.
- **Errors**: standard HTTP status + JSON `{ "message": ... }` (not RFC 9457). 401 = refresh token; 403 = caller lacks access; 404 = id not found or inaccessible.
- **Idempotency**: reads are safe to retry; no idempotency-key mechanism is documented.
