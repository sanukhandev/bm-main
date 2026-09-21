# Database Guidelines

## Database

MySQL is the source-of-record database.

---

## Primary Keys

Use the project's established primary-key strategy.

If using numeric IDs internally, consider public UUID/ULID identifiers only if needed; do not introduce them casually across an existing system.

---

## Branch IDs

Operational tables should generally include:

```text
branch_id
```

with:

- foreign key;
- not-null where branch ownership is mandatory;
- index;
- composite indexes based on common query patterns.

Examples:

```text
(branch_id, status)
(branch_id, created_at)
(branch_id, agreement_no)
(branch_id, property_id)
```

---

## Money

Use:

```text
DECIMAL(precision, scale)
```

Never use FLOAT/DOUBLE for money.

Example:

```text
DECIMAL(18, 2)
```

Precision and currency rules are product decisions.

---

## Enumerations

Use database strings backed by application enums, or database enums if the project has intentionally chosen them.

Application enum examples:

```text
CustomerType
PropertyType
AgreementStatus
PaymentMode
PaymentStatus
WorkOrderStatus
StockMovementType
```

---

## Foreign Keys

Use foreign keys for core referential integrity where compatible with deployment strategy.

Important relationships include:

- property → owner customer;
- component → property;
- agreement → customer;
- agreement asset → property/component;
- payment → agreement/installment;
- receipt → payment;
- work-order item → inventory item;
- purchase-order item → inventory item.

---

## Soft Delete

Use soft delete only where semantically useful.

Do not soft-delete posted accounting records merely to hide them.

For financial records prefer:

```text
void/cancelled/reversed
```

For master data prefer:

```text
active/inactive
```

when historical references must remain valid.

---

## Unique Constraints

Examples:

```text
(branch_id, property_code)
(branch_id, agreement_no)
(branch_id, work_order_no)
(branch_id, purchase_order_no)
(branch_id, invoice_no)
```

Receipt number uniqueness must match the selected numbering scope.

---

## Migrations

Each schema change must be delivered through Laravel migrations.

Migrations should:

- be deploy-safe where practical;
- preserve existing data;
- add indexes intentionally;
- avoid long blocking table rewrites on large production tables where possible;
- provide a safe rollback where feasible.

---

## Seeders

Seed:

- reference data;
- development fixtures;
- required system roles/permissions.

Do not seed production secrets.

---

## Data History

Use explicit history tables for data that requires temporal traceability, such as:

- agreement status changes;
- ownership changes;
- stock movement;
- receipt posting;
- high-value audit events.

Do not rely only on `updated_at` for audit history.
