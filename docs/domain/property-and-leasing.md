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

A Property is not available for a tenant agreement when there is an
overlapping active/reserved tenant agreement for that Property, unless
overlap is explicitly permitted.

Recommended overlap rule for inclusive dates:

```text
new_start <= existing_end
AND
new_end >= existing_start
```

If checkout/end dates are treated as exclusive, use the corresponding exclusive rule consistently.

---

## Agreement Coverage

Owner agreement coverage is stored against one or more Property records.

Tenant agreements must not lease an asset outside Baithul Madeena's valid owner-agreement coverage for the relevant period, if owner-agreement coverage is the source of leasing authority.

---

## Agreement Lifecycle

Suggested owner and tenant agreement statuses:

```text
draft
active
expired
terminated
cancelled
```

Possible transition model:

```text
draft → active
draft → cancelled
active → expired
active → terminated
active → cancelled    # only if business rules allow
```

Activation should validate:

- parties are active;
- asset belongs to branch;
- dates are valid;
- asset is covered;
- tenant asset is available;
- payment schedule reconciles;
- mandatory fields are complete.

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
