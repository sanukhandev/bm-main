# Baithul Madeena API Schema Library

This document is the frontend-facing contract for the REST API. It records the
implemented request and response schemas, security rules, error format, and
conventions required for new endpoints.

The backend is authoritative. Frontend types, services, guards, and forms must
follow this document and must not treat frontend filtering as authorization.

## API identity

| Item | Contract |
| --- | --- |
| Base path | `/api/v1` |
| Authentication | Laravel Sanctum stateful session cookie |
| CSRF endpoint | `GET /sanctum/csrf-cookie` |
| Branch header | `X-Branch-Id` on branch-scoped requests |
| Request ID | `X-Request-Id` response header and error-body field |
| Default page size | `25` |
| Maximum page size | `100` |
| Date/time format | ISO-8601 timestamps; agreement dates are date-only strings |

## Browser security flow

The Angular application must use credentials/cookies on API requests:

```text
GET /sanctum/csrf-cookie
  ↓
POST /api/v1/auth/login
  ↓
GET /api/v1/auth/me
  ↓
GET /api/v1/auth/branches
  ↓
branch-scoped requests include X-Branch-Id
```

Rules:

- Do not store bearer tokens in `localStorage`, `sessionStorage`, or IndexedDB.
- Send cookies with requests (`withCredentials: true` in Angular).
- Send the decoded `XSRF-TOKEN` cookie as the `X-XSRF-TOKEN` header for state-changing requests.
- Do not disable CSRF protection.
- Production CORS origins are explicit; wildcard origins are not allowed with credentials.
- The frontend must never send passwords, session IDs, or CSRF tokens to logs or analytics.

## Common headers

```http
Accept: application/json
Content-Type: application/json
Origin: https://frontend.example.com
X-XSRF-TOKEN: <decoded XSRF-TOKEN cookie>
X-Branch-Id: <authorized branch id>
```

`X-Branch-Id` is required for branch-owned resources. It is a claim only; the
backend verifies the branch against the authenticated user on every request.

## Response envelopes

### Single resource

```json
{
  "data": {
    "id": 1
  }
}
```

### Collection

```json
{
  "data": []
}
```

### Paginated collection

Laravel pagination fields are preserved:

```json
{
  "data": [],
  "links": {
    "first": "...",
    "last": "...",
    "prev": null,
    "next": "..."
  },
  "meta": {
    "current_page": 1,
    "from": 1,
    "last_page": 1,
    "per_page": 25,
    "to": 10,
    "total": 10
  }
}
```

The frontend must use `data`, `links`, and `meta`; it must not expect a custom
`success` wrapper.

## Error model

All `/api/*` errors use this shape:

```json
{
  "message": "Resource not found.",
  "code": "RESOURCE_NOT_FOUND",
  "request_id": "018f..."
}
```

Validation errors add field errors:

```json
{
  "message": "Validation failed.",
  "code": "VALIDATION_ERROR",
  "errors": {
    "email": ["The email field is required."]
  },
  "request_id": "018f..."
}
```

| HTTP | Code | Frontend meaning |
| ---: | --- | --- |
| 400 | `BAD_REQUEST` / `BRANCH_CONTEXT_REQUIRED` | Request or branch context is missing/invalid |
| 401 | `AUTHENTICATION_REQUIRED` / `INVALID_CREDENTIALS` / `ACCOUNT_INACTIVE` | Login or session is not valid |
| 403 | `FORBIDDEN` | Authenticated but not allowed to perform the action |
| 404 | `RESOURCE_NOT_FOUND` / `BRANCH_NOT_FOUND` | Missing or intentionally hidden resource |
| 405 | `METHOD_NOT_ALLOWED` | HTTP method is unsupported |
| 409 | `RESOURCE_CONFLICT` or domain conflict code | Operation conflicts with current state |
| 422 | `VALIDATION_ERROR` | Request fields failed validation |
| 429 | `TOO_MANY_REQUESTS` | Rate limit exceeded |
| 500 | `INTERNAL_SERVER_ERROR` | Safe generic server failure |

Frontend code must branch on `code`, not English `message`. Never display SQL,
stack traces, exception names, or database details.

## Implemented endpoint schemas

### `POST /api/v1/auth/login`

Authentication is not required. CSRF initialization is required for browser
clients.

Request:

```json
{
  "email": "user@example.com",
  "password": "password"
}
```

Rules:

- `email`: required, valid email, normalized to lowercase and trimmed.
- `password`: required string.
- Only users with `status=active` can authenticate.
- Unknown, incorrect, and inactive credentials return the same public response.

Success: `200`

```json
{
  "data": {
    "id": 1,
    "name": "User Name",
    "email": "user@example.com",
    "roles": ["super_admin"],
    "branches": [
      {
        "id": 1,
        "code": "DXB",
        "name": "Dubai",
        "timezone": "Asia/Dubai",
        "currency_code": "AED",
        "status": "active"
      }
    ]
  }
}
```

Never returned: password, remember token, session ID, raw pivot records, or
authentication internals.

### `POST /api/v1/auth/logout`

Authentication and CSRF are required.

Success: `204 No Content`. The session is invalidated and its CSRF token is
regenerated.

### `GET /api/v1/auth/me`

Authentication required; no branch header required.

Returns the same `User` resource shape as login.

### `GET /api/v1/auth/branches`

Authentication required; no branch header required.

Branch Admin users receive active branches from active memberships. Super Admin
users receive all active branches.

Response:

```json
{
  "data": [
    {
      "id": 1,
      "code": "DXB",
      "name": "Dubai",
      "timezone": "Asia/Dubai",
      "currency_code": "AED",
      "status": "active"
    }
  ]
}
```

### `GET /api/v1/customers`

Authentication, active account, and `X-Branch-Id` are required.

Query parameters:

| Parameter | Type | Rules |
| --- | --- | --- |
| `page` | integer | Laravel pagination; defaults to 1 |
| `per_page` | integer | Defaults to 25; maximum 100 |
| `search` | string | Searches supported customer identity fields |
| `status` | string | `active`, `inactive`, or `archived` |
| `customer_type` | string | `individual` or `organization` |
| `sort` | string | Whitelisted fields; `-` prefix means descending |

Customers are always filtered to the verified branch. The response is a
paginated `Customer` collection.

### `POST /api/v1/customers`

Authentication, active account, branch context, authorization, and CSRF are
required.

Request:

```json
{
  "customer_code": "CUS-001",
  "customer_type": "individual",
  "display_name": "Customer Name",
  "legal_name": null,
  "phone": "+971500000000",
  "email": "customer@example.com",
  "tax_registration_no": null,
  "identity_no": null,
  "company_registration_no": null,
  "address_line_1": null,
  "address_line_2": null,
  "city": "Dubai",
  "state_or_emirate": "Dubai",
  "country_code": "AE",
  "notes": null,
  "metadata_json": null
}
```

`branch_id`, `status`, roles, audit actors, and calculated values are
server-controlled and must not be submitted as authorization instructions.

Success: `201` with a `Customer` resource.

### `GET /api/v1/customers/{customer}`

Authentication, active account, and `X-Branch-Id` are required. The customer
must belong to the verified branch. Another branch's ID returns `404` without
revealing that it exists.

Customer response fields:

```json
{
  "data": {
    "id": 1,
    "branch_id": 1,
    "customer_code": "CUS-001",
    "customer_type": "individual",
    "display_name": "Customer Name",
    "legal_name": null,
    "phone": "+971500000000",
    "email": "customer@example.com",
    "tax_registration_no": null,
    "identity_no": null,
    "company_registration_no": null,
    "address_line_1": null,
    "address_line_2": null,
    "city": "Dubai",
    "state_or_emirate": "Dubai",
    "country_code": "AE",
    "status": "active",
    "notes": null,
    "roles": [],
    "created_at": "2026-09-22T08:00:00+00:00",
    "updated_at": "2026-09-22T08:00:00+00:00"
  }
}
```

### `PATCH /api/v1/customers/{customer}`

Uses the same branch and authorization rules as show. Editable fields are the
customer profile fields from the create request. `branch_id`, `customer_code`
ownership, status transitions, roles, and authorization fields are not freely
editable through this endpoint.

Success: `200` with a `Customer` resource.

### `DELETE /api/v1/customers/{customer}`

Authentication, active account, branch context, authorization, and CSRF are
required. This is a safe delete: the row remains and its status becomes
`archived`. The endpoint never physically deletes a customer.

Success: `204 No Content`.

### Property resources

The same CRUD pattern is available under `/api/v1/properties`:

```text
GET    /api/v1/properties
POST   /api/v1/properties
GET    /api/v1/properties/{property}
PATCH  /api/v1/properties/{property}
DELETE /api/v1/properties/{property}
```

All routes require authentication, an active account, and `X-Branch-Id`.

Property create request:

```json
{
  "owner_customer_id": 10,
  "property_code": "FLAT-101",
  "unit_number": "101",
  "property_type": "apartment",
  "name": "Flat 101",
  "building_name": "Building A",
  "address_line_1": "Main Street",
  "city": "Dubai",
  "state_or_emirate": "Dubai",
  "country_code": "AE",
  "area": "85.5000",
  "notes": null,
  "metadata_json": {}
}
```

`owner_customer_id` must identify an active owner customer in the selected
branch. The server sets `branch_id` and `status=active`. Property ownership is
immutable through ordinary PATCH once the property exists; use a dedicated
future ownership workflow if the business permits ownership changes.

Property response:

```json
{
  "data": {
    "id": 20,
    "branch_id": 1,
    "owner_customer_id": 10,
    "owner": { "data": { "id": 10, "display_name": "Owner" } },
    "property_code": "FLAT-101",
    "unit_number": "101",
    "property_type": "apartment",
    "name": "Flat 101",
    "building_name": "Building A",
    "address_line_1": "Main Street",
    "address_line_2": null,
    "city": "Dubai",
    "state_or_emirate": "Dubai",
    "country_code": "AE",
    "area": "85.5000",
    "status": "active",
    "notes": null,
    "metadata_json": {},
    "created_at": "2026-09-22T08:00:00+00:00",
    "updated_at": "2026-09-22T08:00:00+00:00"
  }
}
```

Property list filters are `search`, `status`, `property_type`,
`owner_customer_id`, `page`, `per_page`, and whitelisted `sort` values.

`DELETE /api/v1/properties/{property}` is a safe delete. It sets
`status=archived` and returns `204`; it never removes the property row or its
historical relationships.

### Owner Agreement resources

```text
GET    /api/v1/owner-agreements
POST   /api/v1/owner-agreements
GET    /api/v1/owner-agreements/{owner_agreement}
PATCH  /api/v1/owner-agreements/{owner_agreement}
DELETE /api/v1/owner-agreements/{owner_agreement}
```

Create request:

```json
{
  "agreement_no": "OA-2026-001",
  "owner_customer_id": 10,
  "property_ids": [20, 21],
  "start_date": "2026-01-01",
  "end_date": "2026-12-31",
  "total_amount": "12000.00",
  "currency_code": "AED",
  "payment_count": 12,
  "payment_frequency": "monthly",
  "payment_mode": "bank_transfer",
  "terms_text": null,
  "notes": null
}
```

The owner must have the `owner` business role in the selected branch. Each
property must belong to that same owner and branch. New agreements always start
as `draft`; `branch_id`, status, audit actors, timestamps, and lock version are
server-controlled.

Owner Agreement response fields:

```text
id, branch_id, agreement_no, owner_customer_id, owner, properties,
start_date, end_date, total_amount, currency_code, payment_count,
payment_frequency, payment_mode, terms_text, notes, status, lock_version,
terminated_at, termination_reason, created_at, updated_at
```

Agreement PATCH is allowed only while the status is `draft` or
`pending_approval`. `approved`, `commenced`, `expired`, and `terminated`
records require dedicated lifecycle actions, not ordinary CRUD updates.

Agreement list filters are `search`, `status`, `party_customer_id`, `page`,
`per_page`, and whitelisted `sort` values.

`DELETE /api/v1/owner-agreements/{owner_agreement}` is a safe termination, not
a physical delete. Optional request body:

```json
{ "reason": "Owner record closed" }
```

The agreement becomes `terminated`, termination actor/time/reason are recorded,
status history is appended, and the updated resource is returned with `200`.

### Tenant Agreement resources

```text
GET    /api/v1/tenant-agreements
POST   /api/v1/tenant-agreements
GET    /api/v1/tenant-agreements/{tenant_agreement}
PATCH  /api/v1/tenant-agreements/{tenant_agreement}
DELETE /api/v1/tenant-agreements/{tenant_agreement}
```

Create request:

```json
{
  "agreement_no": "TA-2026-001",
  "tenant_customer_id": 30,
  "properties": [
    {
      "property_id": 20,
      "source_owner_agreement_id": 5
    }
  ],
  "start_date": "2026-01-01",
  "end_date": "2026-12-31",
  "total_amount": "24000.00",
  "currency_code": "AED",
  "payment_count": 12,
  "payment_frequency": "monthly",
  "payment_mode": "cash",
  "terms_text": null,
  "notes": null
}
```

The tenant must have the `tenant` business role in the selected branch. Every
property must be covered by the referenced Owner Agreement in the same branch;
the source agreement/property relationship is enforced by database foreign
keys. New agreements start as `draft`.

Tenant Agreement response fields match Owner Agreements except that the party
fields are `tenant_customer_id` and `tenant`; each property retains its source
owner-agreement relationship in the database.

Tenant PATCH is allowed only for `draft` and `pending_approval` agreements.
DELETE safely terminates the agreement, appends status history, and returns
the updated resource with `200`; it never hard-deletes contractual history.

## Roles and branch security

Current application roles:

```text
super_admin   global role; may select any active branch
branch_admin  branch role; may select only an active assigned branch
```

Branch resolution order:

```text
session authentication
  → active-user check
  → X-Branch-Id validation
  → active branch lookup
  → membership or super-admin authorization
  → request-scoped BranchContext
  → branch-safe route binding
  → policy authorization
```

Missing `X-Branch-Id` returns `400 BRANCH_CONTEXT_REQUIRED`. An inaccessible,
inactive, malformed, or nonexistent branch returns `404 BRANCH_NOT_FOUND`.

The frontend must refresh branch-sensitive data after switching branches and
must never assume that a previously loaded resource remains valid in the new
branch.

## Rules for new endpoints

Every new endpoint must document:

- HTTP method and `/api/v1` path;
- authentication and CSRF requirements;
- whether `X-Branch-Id` is required;
- request body, query, and path parameter schemas;
- server-controlled fields that clients must not provide;
- response resource fields and status code;
- pagination, filters, and sort whitelist;
- error codes and validation fields;
- policy and branch authorization behavior;
- request/response examples;
- feature tests for unauthorized and cross-branch access.

New successful resources must use Laravel Resource-style `data` envelopes.
New errors must preserve `message`, uppercase `code`, and `request_id`.

## Contract ownership

This file belongs to the parent repository so frontend work can consume one
versioned contract independent of backend implementation details. Backend API
changes must update this file in the same change. Breaking changes require
explicit approval and a versioning decision.
