# Testing Strategy

## Testing Pyramid

Use:

- unit tests for calculations/domain rules;
- feature/integration tests for APIs and database behavior;
- frontend unit/component tests for UI logic;
- end-to-end tests for critical workflows.

---

## Mandatory Backend Test Areas

### Branch isolation

For every major branch-scoped resource:

```text
same branch → permitted when role allows
different branch → denied/not found
super admin → permitted when intended
```

Test read and write operations.

### Customers

- individual creation;
- organization creation;
- owner role;
- tenant role;
- both roles.

### Properties

- create each property type;
- accept all supported PropertyType values;
- reject unsupported property types;
- reject cross-branch owner;
- archive with history preserved.

### Owner agreements

- create;
- activate;
- installment schedule;
- invalid date range;
- wrong-owner property;
- termination/expiry.

### Tenant agreements

- create;
- activate;
- reject unavailable asset;
- overlap boundaries;
- cross-branch asset;
- agreement expiry/termination.

Availability tests must cover inclusive boundary conflicts, contained ranges,
non-blocking draft/terminal statuses, multi-property atomicity, owner
agreement date coverage, update self-exclusion, and the branch-scoped
`/api/v1/properties/available` query. Final create/update tests must exercise
the backend recheck under the Property-row locking transaction.

### Agreement lifecycle

Owner and Tenant Agreement tests must cover the shared transition matrix,
locked commercial editing after approval, explicit lifecycle actions,
commencement/expiry processing, hold/resume, cancellation versus termination,
extension, renewal independence, branch isolation, and preservation of payment
history. The scheduled lifecycle processor must be safe to run repeatedly.

### Payments

- cash;
- cheque;
- bank transfer;
- partial payment;
- duplicate prevention;
- receipt generation;
- void/reversal.

### Inventory

- purchase receipt increases stock;
- work-order use decreases stock;
- return increases stock;
- scrap decreases stock;
- cannot over-return;
- insufficient stock behavior.

### Work orders

- property-level;
- direct Property relationship;
- service charge;
- item consumption;
- completion.

### Invoice

- create header;
- line totals;
- branch scope;
- no mandatory links to operational domains.

---

## Frontend Test Areas

- branch selector behavior;
- branch cache invalidation;
- route guard UX;
- property-type dynamic forms;
- agreement installment validation;
- unavailable asset messaging;
- financial double-submit prevention;
- inventory movement views;
- API error rendering.

---

## End-to-End Critical Paths

### Owner onboarding

```text
Create owner
→ Create property
→ Create owner agreement
→ Activate agreement
```

### Tenant leasing

```text
Create tenant
→ Find available asset
→ Create tenant agreement
→ Generate payment schedule
→ Activate agreement
→ Record payment
→ Generate cash receipt
```

### Owner payout

```text
Find due owner installment
→ Record outward payment
→ Generate purchase receipt
```

### Maintenance

```text
Create work order
→ Add service charge
→ Consume inventory
→ Return unused inventory
→ Complete work order
```

### Procurement

```text
Create vendor
→ Create purchase order
→ Approve
→ Receive items
→ Verify stock movement
```

---

## Test Data

Factories should create branch-aware data explicitly.

Avoid factories that silently attach every record to one default branch, as this hides isolation defects.

---

## Regression Rule

Any production bug involving:

- unauthorized access;
- wrong branch data;
- payment totals;
- duplicate receipts;
- availability;
- inventory balance

must receive a regression test.
