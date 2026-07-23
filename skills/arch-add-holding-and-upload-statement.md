---
name: Add a holding and upload its statement
description: Create a fund, company, or account holding in Arch and attach a statement or cash-flow notice file to it.
api: openapi/arch-client-api-openapi.json
operations:
  - POST /client-api/v0/auth/token
  - create-funds-holding
  - create-company-holding
  - create-account-holding
  - get-list-holding-file-types
  - post-holding-statement-upload
  - post-holding-cashflow-upload
---

# Add a holding and upload a statement

Use this to record a new private-market investment in Arch and attach its documents.

## Steps

1. **Authenticate** — `POST /client-api/v0/auth/token` (see the authenticate skill); send `authorization: Bearer <token>` on every call.
2. **Create the holding** — pick the right constructor for the asset type:
   - `create-funds-holding` (`POST /client-api/v0/holdings/fund`) — funds or real estate.
   - `create-company-holding` (`POST /client-api/v0/holdings/company`) — direct company investments.
   - `create-account-holding` (`POST /client-api/v0/holdings/account`) — bank/brokerage accounts.
   A holding links an Investing Entity to an Issuing Entity via one of the Issuing Entity's Offerings.
3. **Check accepted file types** — `get-list-holding-file-types` (`GET /client-api/v0/holdings/{holdingId}/files/types`) before uploading.
4. **Upload documents** — `post-holding-statement-upload` (`POST /client-api/v0/holdings/{id}/files/statements`) for statements, or `post-holding-cashflow-upload` (`POST /client-api/v0/holdings/{id}/files/cash-flows`) for cash-flow notices. Use `post-holding-single-file-upload` for a generic file.

## Rules
- **Currency**: cash-flow creation assumes USD unless the payload specifies otherwise.
- **Writes have consequences**: creating holdings and uploading files mutate the account of record — confirm the target Investing Entity and Issuing Entity first.
- **No idempotency key**: retrying a create may duplicate the holding; verify with a `GET /client-api/v0/holdings` filter before retrying a failed write.
- **Errors**: 400 = payload/schema issue; 403 = no access to the entity; 409 = conflict/duplicate. 429 = rate limited, back off per `RateLimit-Reset`.
