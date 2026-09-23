# Property & Leasing

## Property Types

Supported property types:

```text
apartment
villa
shop
office
space
labor_camp
warehouse
land
```

Each row in `properties` is an independently managed and leasable
real-estate asset. `unit_number` may identify the physical property or
property number, and `area` is a scalar Property attribute. Neither field
creates a child entity.

These eight values are the authoritative PropertyType values for the current
application contract.

---

## Owner Relationships

A property must have an owner customer.

If ownership can change over time, do not simply overwrite history. Introduce ownership history with effective dates.

If multi-owner ownership is required in the future, evolve to an ownership pivot with percentage/share.

---

## Availability

Availability is date-sensitive.

A Property is available only when the requested dates are covered by a valid
Owner Agreement and no blocking Tenant Agreement overlaps the same Property.
The current blocking Tenant Agreement statuses are `pending_approval`,
`approved`, `commenced`, and `on_hold`. Draft, cancelled, terminated, and
expired agreements do not reserve the Property.

Recommended overlap rule for inclusive dates:

```text
new_start <= existing_end
AND
new_end >= existing_start
```

If checkout/end dates are treated as exclusive, use the corresponding exclusive rule consistently.

The backend rechecks this rule inside the Tenant Agreement create/update
transaction. It locks selected Property rows in ascending ID order before
checking overlaps, so an earlier availability response cannot authorize a
conflicting save. `GET /api/v1/properties/available` exposes the same
branch-scoped query for the frontend; it is advisory only.

---

## Agreement Coverage

Owner agreement coverage is stored against one or more Property records.

Tenant agreements must not lease an asset outside Baithul Madeena's valid
owner-agreement coverage for the relevant period. Tenant dates must satisfy:

```text
tenant_start_date >= owner_start_date
AND
tenant_end_date <= owner_end_date
```

---

## Agreement Lifecycle

Owner and Tenant Agreements use the same backend-authoritative statuses:

```text
draft
pending_approval
approved
commenced
on_hold
expired
terminated
cancelled
```

The normal transition model is:

```text
draft → pending_approval → approved → commenced
draft → cancelled
pending_approval → draft | cancelled
approved → cancelled       # only before commencement
commenced → on_hold | expired | terminated
on_hold → commenced | expired | terminated
```

Activation should validate:

- parties are active;
- asset belongs to branch;
- dates are valid;
- asset is covered;
- tenant asset is available;
- payment schedule reconciles;
- mandatory fields are complete.

Approved and later agreements are not ordinarily editable. Submit, approve,
commence, hold, resume, expire, terminate, cancel, extend, and renew are
explicit lifecycle operations. Extension records the old and new end dates;
renewal creates a new draft agreement with a new backend-generated number and
does not copy payments or receipt history.

The `agreements:process-lifecycle` command runs daily using each branch's
timezone. It commences approved agreements whose start date has arrived and
expires commenced/on-hold agreements after their end date. It is idempotent.

---

## Payment Terms

Store structured data where possible.

Example:

```text
payment_count = 4
payment_frequency = quarterly
payment_mode = cheque
```

Avoid storing all business-critical payment terms only as free text.

Free-text terms may be stored in addition to structured fields.

---

## Installment Schedule

An agreement may generate installments.

Each installment should include:

```text
installment_no
due_date
amount
payment_mode
status
paid_amount
```

The sum of installment amounts must equal the agreement's scheduled payable amount under defined rounding rules.

---

## Lease Property Reference

Agreement property references use `property_id` through the
`owner_agreement_properties` and `tenant_agreement_properties` relationships.
Multiple Properties per agreement remain supported.

---

## Deactivation and Deletion

Do not hard-delete Properties referenced by:

- agreements;
- payments;
- work orders;
- inventory usage;
- receipts.

Use inactive/archived status.

Historical agreements must continue to resolve their asset descriptions even if the asset later becomes inactive.
