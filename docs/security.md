# Security

## Core Security Boundary

The primary domain security boundary is branch authorization.

Every sensitive query and mutation must be both:

```text
permission authorized
AND
branch authorized
```

---

## IDOR Protection

Never authorize access merely because a valid record ID was supplied.

Example risk:

```text
GET /properties/125
```

The backend must verify that property `125` belongs to a branch the user may access.

Apply the same rule to nested entities and downloads.

---

## Mass Assignment

Do not allow users to mass-assign protected fields such as:

```text
branch_id
created_by
posted_by
receipt_no
status
owner_customer_id where derived
financial totals where server-calculated
```

unless explicitly validated and authorized.

---

## Branch Injection

Treat request-provided branch identifiers as claims, not truth.

Validate against authenticated permissions.

---

## Financial Security

For posted financial records:

- restrict edits;
- log voids;
- preserve numbering;
- require server-side amount validation;
- prevent duplicate submission.

---

## Inventory Security

Restrict:

- adjustments;
- scrap;
- receiving;
- stock transfers if later introduced.

Inventory write-offs should be auditable.

---

## Files

Validate:

- MIME type;
- extension;
- file size;
- malware scanning if infrastructure supports it.

Use authorized download endpoints or signed URLs.

Do not expose private file buckets publicly.

---

## Secrets

Secrets belong in environment/secret management.

Never commit:

```text
database passwords
API keys
JWT secrets
AI provider keys
SMTP credentials
cloud credentials
```

---

## Rate Limiting

Apply rate limits to:

- authentication;
- password reset;
- AI endpoints;
- expensive searches/reports;
- public or externally callable endpoints.

---

## Audit Logging

Sensitive actions should capture enough context to reconstruct what happened.

Audit logs themselves should be access-controlled and protected from ordinary modification.

---

## SQL Injection

Use Eloquent/query builder parameterization.

Whitelist user-selected sort/filter fields.

Never concatenate untrusted SQL fragments.

---

## XSS

Angular's normal escaping should remain enabled.

Do not bypass sanitization for arbitrary customer-entered HTML.

---

## CSRF/CORS

Configure according to the chosen Laravel/Angular authentication model.

Allow only intended origins in production.

---

## Security Tests

Mandatory cases include:

- branch A user cannot read branch B record;
- branch A user cannot update/delete branch B record;
- branch A user cannot reference branch B foreign key;
- normal user cannot enter all-branch mode;
- super admin branch switching works as designed;
- private attachments obey branch rules.
Financial authorization is separated into `accounts.view`, `accounts.post`,
and `accounts.void`. Permission never bypasses branch isolation.

Audit viewing is separately protected by `audit.view`. Audit records are
append-only, branch-scoped, and exclude passwords, tokens, secrets, and raw
authorization material. Critical mutation audit inserts occur in the same
database transaction as the business change where practical; scheduled system
actions use a null actor with `metadata.actor_type = system`.

## Phase 1 security rules

Branch ownership is resolved from authenticated branch context; client-supplied
`branch_id` values are not trusted. Foreign keys are validated against that
branch, lifecycle/status changes use action endpoints, and posted financial
amounts, directions, allocations, numbers, and audit records are
server-authoritative. Angular visibility is UX only; backend middleware and
policies remain the security boundary.

Production deployments must set `APP_DEBUG=false`, use HTTPS with secure,
HttpOnly cookies, an appropriate SameSite policy, explicit frontend origins in
CORS, and trusted-proxy configuration matching the deployment edge. All-branch
access remains an explicit future capability and is never inferred from a
missing branch context.
