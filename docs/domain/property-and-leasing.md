# Property & Leasing

## Property Types

Supported initial types:

| Type | Rentable Child |
|---|---|
| Apartment | Unit |
| Villa | Room |
| Shop | Area |
| Office | Area |
| Space | Area or property-level scope |
| Labor Camp | Bed |
| Warehouse | Area or property-level scope |
| Land | Area or property-level scope |

Apartment unit classifications initially include:

```text
studio
1_bhk
2_bhk
3_bhk
4_bhk
```

These should be configurable/extensible where practical.

---

## Owner Relationships

A property must have an owner customer.

If ownership can change over time, do not simply overwrite history. Introduce ownership history with effective dates.

If multi-owner ownership is required in the future, evolve to an ownership pivot with percentage/share.

---

## Availability

Availability is date-sensitive.

A property/component is not available for a tenant agreement when there is an overlapping active/reserved tenant agreement for the same rentable scope, unless overlap is explicitly permitted.

Recommended overlap rule for inclusive dates:

```text
new_start <= existing_end
AND
new_end >= existing_start
```

If checkout/end dates are treated as exclusive, use the corresponding exclusive rule consistently.

---

## Agreement Coverage

Owner agreement coverage may be:

- entire property;
- selected units/components;
- an explicitly modeled scope.

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

## Lease Asset Abstraction

Preferred API behavior is to expose a normalized asset reference.

Example conceptual payload:

```json
{
  "asset_type": "rentable_component",
  "asset_id": 123
}
```

or:

```json
{
  "property_id": 10,
  "rentable_component_id": 123
}
```

Do not expose separate incompatible agreement APIs for every property subtype unless necessary.

---

## Deactivation and Deletion

Do not hard-delete property/components referenced by:

- agreements;
- payments;
- work orders;
- inventory usage;
- receipts.

Use inactive/archived status.

Historical agreements must continue to resolve their asset descriptions even if the asset later becomes inactive.
