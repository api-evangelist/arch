---
name: Triage tasks and retrieve tax documents
description: List open tasks, assign and complete them, and pull tax documents and their files from Arch.
api: openapi/arch-client-api-openapi.json
operations:
  - GET /client-api/v0/tasks
  - PATCH /client-api/v0/tasks/{userId}/assign
  - POST /client-api/v0/tasks/{id}/complete
  - get-list-tax-documents
  - get-tax-document-by-id
  - get-tax-document-download-file
---

# Triage tasks and retrieve tax documents

Use this to work the operational queue and pull tax paperwork for a client.

## Steps

1. **Authenticate** — obtain a Bearer JWT via `POST /client-api/v0/auth/token`.
2. **List tasks** — `GET /client-api/v0/tasks`; filter with `isComplete=false`, `assignedToUsers`, and due-date windows (`afterDueDate` / `beforeDueDate`).
3. **Assign** — `PATCH /client-api/v0/tasks/{userId}/assign` to route one or more tasks to a user.
4. **Complete** — `POST /client-api/v0/tasks/{id}/complete` once the work is done; update context with `PATCH /client-api/v0/tasks/{id}/task-notes`.
5. **Pull tax documents** — `get-list-tax-documents` (`GET /client-api/v0/tax-documents`), then `get-tax-document-by-id` for one, and list files with `get-tax-document-list-files`.
6. **Download files** — `get-tax-document-download-file` (`GET /client-api/v0/tax-documents/{taxDocId}/files/{fileId}/download`) returns the file as originally received.

## Rules
- **Pagination**: tasks and tax documents come back as Pages (`offset`/`limit`, default 25, max 1000).
- **Completing a task is a write** — it changes state for other users; confirm the task id first.
- **Errors**: 404 = task/tax-doc id not found or inaccessible; 403 = no access. On 429 honor `RateLimit-Reset`.
