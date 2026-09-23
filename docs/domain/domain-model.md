# Domain Model

## Core Aggregate Map

```text
Branch
├── Users
├── Customers
│   ├── Owner role
│   └── Tenant role
├── Properties
├── Owner Agreements
├── Tenant Agreements
├── Payments
├── Receipts
├── Work Orders
├── Inventory
├── Vendors
├── Purchase Orders
└── Invoices
```

---

## Customer

A customer represents a person or organization.

Suggested attributes:

```text
id
branch_id
customer_code
customer_type          # individual | organization
display_name
legal_name
phone
email
tax_registration_no
status
notes
```

Individual-specific fields may live in a profile table.

Organization-specific fields may live in a profile table.

Roles may be stored in a role table/pivot:

```text
owner
tenant
```

A customer may have both roles.

---

## Property

A property is an owner-linked real-estate asset managed by Baithul Madeena.

Suggested attributes:

```text
id
branch_id
owner_customer_id
property_code
name
property_type
address
status
notes
```

`property_type` may include:

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

---

## Property as the Leasable Entity

Each Property record is the independently managed and leasable real-estate
asset. Property type describes the record; it does not create a child asset
hierarchy.

Supported `property_type` values are:

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

`unit_number` and `area` are scalar Property attributes. They do not refer
to separate records.

---

## Owner Agreement

Represents the contractual relationship where Baithul Madeena takes management/lease rights from an owner.

Suggested structure:

```text
OwnerAgreement
├── OwnerAgreementAsset
├── OwnerAgreementInstallment
├── Payments
└── Receipts / outward financial documents
```

Key fields:

```text
id
branch_id
agreement_no
owner_customer_id
start_date
end_date
payment_terms
payment_count
payment_mode
total_amount
status
```

---

## Tenant Agreement

Represents the contractual relationship where Baithul Madeena leases an asset to a tenant.

Suggested structure:

```text
TenantAgreement
├── TenantAgreementAsset
├── TenantAgreementInstallment
├── Payments
└── CashReceipts
```

Key fields:

```text
id
branch_id
agreement_no
tenant_customer_id
start_date
end_date
payment_terms
payment_count
payment_mode
total_amount
status
```

---

## Payment Schedule

Both owner and tenant agreements can use payment schedules.

Suggested common fields:

```text
id
branch_id
agreement_type
agreement_id
installment_no
due_date
amount
status
paid_amount
```

Do not use polymorphism automatically if it harms referential integrity. Separate tables may be preferable.

---

## Payment

A payment represents actual settlement against a scheduled or ad-hoc amount.

Suggested fields:

```text
id
branch_id
direction             # inward | outward
payment_mode          # cash | cheque | bank_transfer
reference_no
transaction_date
amount
status
payer/payee context
notes
```

---

## Receipt

Recommended separation:

```text
CashReceipt       # inward
PurchaseReceipt   # outward
```

Receipt numbering should be unique under a documented scope such as:

```text
global
per branch
per financial year
per branch + financial year
```

Pick one and enforce it at database level where possible.

---

## Work Order

Suggested structure:

```text
WorkOrder
├── WorkOrderServiceCharge
├── WorkOrderItem
└── Status History
```

Core fields:

```text
id
branch_id
property_id
work_order_no
description
status
service_charge
opened_at
completed_at
```

---

## Inventory

Suggested model:

```text
InventoryItem
├── InventoryBalance
└── StockMovement
```

Stock movements are the source of truth.

Possible movement types:

```text
purchase_in
work_order_out
work_order_return
vendor_return
adjustment_in
adjustment_out
scrap
```

---

## Vendor

Suggested fields:

```text
id
branch_id
vendor_code
name
phone
email
tax_registration_no
address
status
```

Decide explicitly whether vendors are branch-local or globally shared.

---

## Purchase Order

Suggested structure:

```text
PurchaseOrder
├── PurchaseOrderItem
└── GoodsReceipt / Receiving Event
```

Core fields:

```text
id
branch_id
vendor_id
purchase_order_no
order_date
status
subtotal
tax
total
notes
```

Receiving inventory should generate stock movements.

---

## Invoice

The invoice module is independent for now.

Suggested structure:

```text
Invoice
├── InvoiceLine
```

Header:

```text
id
branch_id
invoice_no
invoice_date
customer_name_text
customer_address_text
notes
subtotal
tax
total
status
```

Line:

```text
description
quantity
unit_price
tax
line_total
```

Current rule: do not require foreign-key relationships to operational modules.

---

## Dashboard Read Model

At minimum:

```text
owner_count
tenant_count
property_count
active_owner_agreement_count
active_tenant_agreement_count
```

Dashboard should be treated as a read model/aggregate query, not as manually maintained counters unless there is a proven performance need.
