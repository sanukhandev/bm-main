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
