# API Conventions

## Base

Recommended versioning:

```text
/api/v1/...
```

---

## Authentication

Use the backend project's chosen Laravel-compatible authentication strategy.

Every protected endpoint must derive user identity from authentication, not request payload.

---

## Branch Context

Recommended options:

```text
X-Branch-Id header
```

or a server-side selected active branch.

If an `X-Branch-Id` header is used:

- normal users may specify only an authorized branch;
- super admin may specify any valid branch;
- the backend validates it on every request;
- absence behavior must be deterministic.

For all-branch reporting, use explicit endpoints/query semantics rather than treating missing branch as "all."

---

## Resource Routes

Examples:

```text
GET    /api/v1/customers
POST   /api/v1/customers
GET    /api/v1/properties
POST   /api/v1/properties
GET    /api/v1/owner-agreements
POST   /api/v1/owner-agreements
POST   /api/v1/owner-agreements/{id}/activate
GET    /api/v1/tenant-agreements
POST   /api/v1/tenant-agreements
POST   /api/v1/tenant-agreements/{id}/activate
POST   /api/v1/payments
POST   /api/v1/accounts/transactions/{id}/void
GET    /api/v1/work-orders
POST   /api/v1/work-orders
POST   /api/v1/purchase-orders/{id}/receive
POST   /api/v1/inventory/scrap
GET    /api/v1/dashboard/operational
```

Use action endpoints for meaningful state transitions rather than generic status patching when the transition has business side effects.

---

## Response Envelope

Use one consistent convention.

Example:

```json
{
  "data": {},
  "meta": {}
}
```

Validation/error example:

```json
{
  "message": "Validation failed.",
  "errors": {
    "field": ["The field is required."]
  },
  "code": "VALIDATION_ERROR"
}
```

---

## Pagination

All potentially large collections should be paginated.

Example query:

```text
?page=1&per_page=25
```

Cap `per_page` to prevent abuse.

---

## Filtering

Prefer explicit filters:

```text
?status=active
?customer_type=individual
?role=owner
?property_type=apartment
?search=...
?date_from=...
?date_to=...
```

Do not expose arbitrary SQL-like filter expressions unless intentionally designed.

---

## Sorting

Whitelist sortable fields.

Example:

```text
?sort=-created_at
```

Do not pass raw user-provided column names directly into SQL ordering without validation.

---

## Idempotency

For sensitive create/post operations, support idempotency where practical.

Possible header:

```text
Idempotency-Key
```

Persist enough data to reject accidental duplicate posting.

---

## Optimistic Concurrency

For high-conflict records, consider:

```text
updated_at
version
```

and reject stale writes when necessary.

---

## Status Codes

Common usage:

```text
200 success
201 created
204 no content
400 malformed/business request where appropriate
401 unauthenticated
403 unauthorized
404 not found or inaccessible resource
409 business conflict
422 validation error
```

Be careful not to reveal existence of inaccessible cross-branch records.

---

## API Documentation

Maintain OpenAPI/Swagger documentation if the project adopts it.

Document:

- branch context;
- permissions;
- status transitions;
- enums;
- error codes;
- examples.

## Current API foundation

The backend uses Laravel Sanctum's stateful SPA authentication with the existing
session guard. The Angular client first requests `/sanctum/csrf-cookie`, then
logs in through `POST /api/v1/auth/login` with cookies and CSRF enabled. It must
send `X-Branch-Id` on branch-scoped requests; the server verifies that claim
against the authenticated user's active membership or global `super_admin`
role. Super admins still operate inside the explicitly selected branch.

Successful resources use Laravel JSON Resource responses. API failures use:

```json
{
  "message": "Resource not found.",
  "code": "RESOURCE_NOT_FOUND",
  "request_id": "..."
}
```

Validation failures additionally include an `errors` object. API requests carry
an `X-Request-Id` response header and the same ID in error bodies.
Financial routes require `accounts.view` for reads, `accounts.post` for new
postings and cheque transitions, and `accounts.void` for voiding posted
transactions, in addition to authenticated branch context.
