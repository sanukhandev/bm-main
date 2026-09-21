# Backend Guidelines

## Stack

```text
Laravel
PHP
MySQL
REST/JSON
```

Use the versions declared by the backend project as authoritative.

---

## Controllers

Controllers should:

- receive validated requests;
- invoke application/domain services;
- return resources/responses.

Controllers should not contain large business workflows.

---

## Validation

Use Laravel Form Requests where appropriate.

Validate:

- type;
- required/nullable;
- enum values;
- date order;
- numeric bounds;
- branch-safe foreign keys;
- state-dependent rules.

Never use a raw `exists:table,id` rule alone for branch-sensitive references if that could validate an ID from another branch.

---

## Authorization

Use policies/gates/services to enforce permissions.

Authorization questions include both:

```text
Can this user perform this action?
Does this record belong to a branch the user can access?
```

Both must pass.

---

## Branch Scope

Centralize branch scoping.

Possible approaches:

- explicit query scopes;
- repository filters;
- context-aware global scopes;
- a branch-aware query service.

If global scopes are used, design carefully for:

- CLI/jobs;
- super admin;
- migrations/seeding;
- cross-branch reports;
- background workers.

Do not scatter `where('branch_id', ...)` inconsistently throughout controllers.

---

## Services / Actions

Use a service/action when a use case has:

- multiple models;
- transactions;
- state transitions;
- stock movement;
- payment posting;
- receipt generation;
- audit logging.

Examples:

```text
ActivateOwnerAgreement
ActivateTenantAgreement
PostTenantPayment
PostOwnerPayment
ReceivePurchaseOrder
ConsumeInventoryForWorkOrder
ReturnWorkOrderInventory
ScrapInventory
SwitchActiveBranch
```

---

## Database Transactions

Wrap multi-write business operations:

```php
DB::transaction(function () {
    // validate state
    // create/update records
    // stock/payment/receipt changes
    // audit
});
```

External network calls should generally not be made inside a long-running database transaction.

---

## Eloquent

Use:

- relationships;
- scopes;
- eager loading;
- casts;
- enums;
- resource transformers.

Avoid:

- hidden N+1 queries;
- unbounded `get()` for large tables;
- arbitrary mass assignment;
- business-critical logic only in accessors.

---

## API Resources

Use API Resources or a consistent transformer layer.

Do not expose internal fields accidentally.

Sensitive fields include:

- internal audit metadata;
- deleted markers;
- secret integration identifiers;
- private file paths.

---

## Exceptions

Use meaningful domain exceptions where helpful.

Examples:

```text
AssetNotAvailableException
AgreementNotActiveException
InsufficientInventoryException
UnauthorizedBranchException
PaymentAlreadyPostedException
InvalidStatusTransitionException
```

Map exceptions to consistent API errors.

---

## Jobs

Queued jobs must carry enough branch context to operate safely.

Never assume an interactive user's active branch still exists in worker memory.

Persist explicit:

```text
branch_id
actor_user_id where needed
entity IDs
```

and re-authorize/re-validate as appropriate.

---

## Logging

Structured logs should include:

```text
request_id
user_id
branch_id
entity identifiers
error code
```

Never log passwords, tokens, full payment credentials, or sensitive identity documents.

---

## Testing

Use feature tests for:

- authorization;
- validation;
- branch scope;
- state transitions;
- financial posting;
- inventory movement.

Use unit tests for isolated calculations and domain rules.
