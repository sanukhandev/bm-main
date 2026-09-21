# Frontend Guidelines

## Stack

```text
Angular
TypeScript
```

Use the Angular version declared in the frontend submodule.

---

## Branch Context

Branch context is a first-class frontend state.

The application should expose:

- current branch;
- available branches;
- whether the user can switch branches;
- whether "all branches" mode is active for supported super-admin views.

When branch changes:

1. update context;
2. clear/invalidate branch-sensitive caches;
3. reload current feature data;
4. avoid displaying stale previous-branch data.

---

## Authorization

Frontend guards improve UX but are not security controls.

Use:

- route guards;
- feature visibility;
- permission-aware action buttons.

The backend remains authoritative.

---

## API Layer

Centralize HTTP access in feature/domain services.

Avoid direct `HttpClient` calls scattered across many components.

Recommended concerns:

```text
authentication interceptor
branch-context interceptor
error interceptor
API services
typed DTOs
```

---

## Models

Keep API DTOs and UI view models explicit.

Avoid widespread `any`.

Use enums/unions for stable values such as:

```text
payment modes
agreement statuses
property types
customer types
```

---

## Forms

Use reactive forms for complex ERP forms.

Agreement forms should validate:

- customer;
- asset;
- date range;
- total;
- installment count;
- installment total;
- payment mode.

Financial forms should prevent accidental repeat submission.

---

## Listings

ERP list pages should support:

- pagination;
- search;
- filters;
- sorting;
- empty state;
- loading state;
- error state;
- permission-aware actions.

Persist filters in URL query parameters where useful.

---

## Dashboard

Dashboard cards:

```text
Owners
Tenants
Properties
Active Owner Agreements
Active Tenant Agreements
```

Each dashboard request must reflect current branch context.

Do not add client-side totals from incomplete paginated datasets.

---

## Property UI

The property form should adapt by property type.

Examples:

```text
Apartment → manage units
Villa → manage rooms
Labor Camp → manage beds
Shop/Office → manage areas
```

Keep shared field logic reusable.

---

## Agreements UI

Agreement screens should clearly show:

- party;
- assets;
- date range;
- status;
- payment terms;
- installment schedule;
- payment history;
- receipt links.

Prevent selection of unavailable tenant assets.

The backend must re-check availability on submit.

---

## Inventory UI

Inventory pages should show:

- item;
- stock on hand;
- movement history;
- reorder indication;
- purchase receipts;
- work-order consumption;
- returns;
- scrap.

Do not provide direct "edit stock quantity" without creating an auditable adjustment workflow.

---

## Error Handling

Map backend errors to actionable user messages.

Examples:

```text
ASSET_NOT_AVAILABLE
INSUFFICIENT_INVENTORY
INVALID_STATUS_TRANSITION
UNAUTHORIZED_BRANCH
PAYMENT_ALREADY_POSTED
```

Do not expose raw stack traces.

---

## Accessibility

Use semantic HTML, labels, keyboard-accessible controls, and proper focus handling.

ERP density is not an excuse to make critical financial screens inaccessible.
