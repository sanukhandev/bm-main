# Baithul Madeena ERP — Agent Guide

## Purpose

Baithul Madeena is a multi-branch real-estate ERP with AI capabilities.

This repository contains the main project and two Git submodules:

```text
baithul-madeena/
├── AGENTS.md
├── docs/
├── backend/      # Laravel + MySQL submodule
└── frontend/     # Angular submodule
```

Any coding agent working in this repository must preserve:

1. strict branch-level data isolation,
2. super-admin cross-branch access,
3. accounting and receipt traceability,
4. property/unit occupancy integrity,
5. inventory stock integrity,
6. agreement/payment consistency,
7. auditability of sensitive actions,
8. backward-compatible API behavior unless a breaking change is explicitly approved.

---

## Product Model

Baithul Madeena manages property leasing operations between property owners and tenants.

The high-level business flow is:

```text
Owner
  ↓
Owner Property / Property Units
  ↓
Owner Agreement with Baithul Madeena
  ↓
Property / Unit made available for leasing
  ↓
Tenant Agreement
  ↓
Tenant Payments / Receipts
```

Maintenance and inventory operate alongside the leasing workflow.

---

## Multi-Branch Rules

The application supports more than one branch.

### Normal branch users

A normal user:

- belongs to one or more explicitly assigned branches;
- operates within one active branch at a time;
- must only see data that belongs to the active branch;
- must never obtain another branch's records by changing an ID, query parameter, payload field, URL, or API request.

### Super admin

A super admin:

- can access every branch;
- can switch the active branch;
- can request consolidated/all-branch reporting where explicitly supported;
- still operates under auditable branch context.

### Non-negotiable implementation rule

Branch isolation must be enforced on the backend.

Frontend filtering is only a UX convenience and must never be treated as an authorization control.

See [Multi-Tenancy & Branch Isolation](docs/architecture.md#branch-isolation).

---

## Core Domains

The primary domains are:

- Branches and users
- Customers
- Owners
- Tenants
- Properties
- Property components / rentable assets
- Owner agreements
- Tenant agreements
- Payments
- Cash receipts
- Purchase receipts
- Maintenance work orders
- Inventory
- Vendors
- Purchase orders
- Inventory returns
- Inventory scrap/write-off
- Standalone invoices
- Dashboard/reporting
- AI features
- Audit logs

See [Domain Model](docs/domain/domain-model.md).

---

## Customer Model

Owners and tenants are represented as customers.

Each customer has:

```text
customer_type:
- individual
- organization
```

A customer can act as:

```text
roles:
- owner
- tenant
- both
```

Do not duplicate the same person or organization into unrelated owner and tenant master tables unless the database design explicitly requires role-extension tables.

---

## Property Model

Supported property types and rentable structures:

| Property Type | Child / Rentable Structure |
|---|---|
| Apartment | Units |
| Villa | Rooms |
| Shop | Areas |
| Office | Areas |
| Space | Area |
| Labor Camp | Beds |
| Warehouse | Area |
| Land | Area |

Apartment units may use classifications such as:

- Studio
- 1 BHK
- 2 BHK
- 3 BHK
- 4 BHK

Design the model so new property types and rentable component types can be added without large schema rewrites.

See [Property & Leasing Model](docs/domain/property-and-leasing.md).

---

## Agreements

### Owner agreement

Baithul Madeena takes a property, or agreed rentable scope, from an owner through an owner agreement.

An owner agreement may define:

- agreement number;
- owner;
- property/properties;
- covered units/components;
- agreement start and end dates;
- payment terms;
- number of payments/installments;
- payment mode;
- installment schedule;
- agreement status;
- financial totals;
- attachments and notes.

### Tenant agreement

Baithul Madeena leases an available property or rentable component to a tenant.

A tenant agreement may define:

- agreement number;
- tenant;
- leased property/component;
- start and end dates;
- payment terms;
- number of payments/installments;
- payment mode;
- installment schedule;
- agreement status;
- financial totals;
- attachments and notes.

### Supported payment modes

```text
cash
cheque
bank_transfer
```

Use enums/constants rather than unvalidated arbitrary strings.

---

## Financial Documents

### Inward revenue

All money received by Baithul Madeena must produce a cash receipt or equivalent receipt record.

Examples:

- tenant rent collection;
- service charges received;
- other approved inward transactions.

### Outward revenue / payment

All money paid out by Baithul Madeena must produce a purchase receipt or corresponding outward-payment document.

Examples:

- owner payment;
- vendor payment where applicable;
- approved operating expense.

Financial documents must be traceable to their originating transaction.

Never silently delete posted financial records. Prefer reversal, void, cancellation, or compensating entries.

---

## Maintenance

A work order may be created for properties or their rentable components.

Each work order can contain:

- property/component;
- request details;
- status;
- assigned employee/vendor;
- service charge;
- inventory items consumed;
- item quantity;
- item cost;
- labor/other charges;
- timestamps;
- notes;
- attachments.

Inventory used by a work order must create auditable stock movements.

See [Maintenance & Inventory](docs/domain/maintenance-inventory.md).

---

## Inventory

Inventory is replenished through purchase orders from vendors.

Supported stock movements include:

- purchase receipt / stock-in;
- work-order consumption;
- return to inventory;
- vendor return where implemented;
- adjustment;
- scrap/write-off.

Never update stock-on-hand as an unexplained number.

Every stock change must be represented by a stock transaction or movement record.

---

## Invoice Module

The invoice module is currently independent.

Invoices:

- contain invoice header/details;
- contain line items;
- are entered independently;
- do not currently require relationships with owners, tenants, agreements, work orders, or other modules.

Do not introduce mandatory relationships without product approval.

The design should still leave room for future optional links.

---

## Dashboard

The operational dashboard should expose at minimum:

- number of owners;
- number of tenants;
- number of properties;
- number of active owner agreements;
- number of active tenant agreements.

All dashboard metrics must respect the current branch context.

For a super admin:

- when a branch is selected, show that branch;
- when "all branches" is explicitly selected, aggregate authorized branches.

Metric definitions must be stable and documented.

---

## Backend Standards

Backend stack:

```text
Laravel
MySQL
REST API
```

Agents working in `backend/` must:

- follow the backend submodule's own `AGENTS.md` if present;
- keep controllers thin;
- place business logic in services/actions/domain classes;
- use Form Requests for validation;
- use Policies/Gates for authorization;
- centralize branch scoping;
- use transactions for multi-record financial or stock workflows;
- avoid N+1 queries;
- use migrations for schema changes;
- use enums/value objects where domain values are constrained;
- create audit records for sensitive changes;
- write tests for branch isolation and business invariants.

See [Backend Guidelines](docs/backend/backend-guidelines.md).

---

## Frontend Standards

Frontend stack:

```text
Angular
TypeScript
```

Agents working in `frontend/` must:

- follow the frontend submodule's own `AGENTS.md` if present;
- keep branch context explicit;
- never use frontend-only authorization;
- centralize API services;
- use typed models/interfaces;
- handle loading, empty, success, and error states;
- avoid duplicating domain rules that belong to the backend;
- invalidate/refetch branch-sensitive cached data after branch switching;
- keep financial forms deterministic and validation-rich.

See [Frontend Guidelines](docs/frontend/frontend-guidelines.md).

---

## API Rules

All branch-scoped resources must derive branch authorization from authenticated context.

Do not trust a user-supplied `branch_id` without authorization.

Preferred behavior:

```text
Authenticated user
  ↓
Resolve authorized active branch
  ↓
Apply branch scope
  ↓
Authorize resource/action
  ↓
Execute query/mutation
```

Super-admin cross-branch endpoints must be explicit.

See [API Conventions](docs/backend/api-conventions.md).

The frontend-facing request, response, error, and security schemas are
maintained in [API Schema Library](docs/api-schema-library.md). Any new or
changed API contract must update that document in the same change.

---

## Data Integrity Invariants

Agents must preserve these invariants:

1. A branch user cannot access unauthorized branch data.
2. A tenant agreement cannot lease an unavailable asset for overlapping dates unless the business explicitly supports overlap.
3. A rentable component belongs to exactly one property.
4. Owner agreement coverage must match the owner/property relationship.
5. Payment schedules must reconcile with agreement totals subject to approved rounding rules.
6. Posted receipts must have immutable/auditable numbering.
7. Inventory cannot change without a stock movement.
8. Work-order consumption cannot silently create inventory.
9. Scrapped stock must be recorded as a write-off movement.
10. Dashboard totals must use the same source-of-truth filters as listing/report APIs.

---

## Naming

Use clear domain names.

Preferred:

```text
OwnerAgreement
TenantAgreement
PaymentSchedule
Payment
CashReceipt
PurchaseReceipt
WorkOrder
WorkOrderItem
InventoryItem
StockMovement
PurchaseOrder
PurchaseOrderItem
Invoice
InvoiceLine
RentableComponent
```

Avoid vague names such as:

```text
Data
Info
Master
TransactionData
Details2
```

unless required by an existing legacy schema.

---

## Status Modeling

Prefer explicit statuses.

Examples:

```text
AgreementStatus:
draft
active
expired
terminated
cancelled

WorkOrderStatus:
draft
open
assigned
in_progress
completed
cancelled

PaymentStatus:
pending
partially_paid
paid
overdue
cancelled

PurchaseOrderStatus:
draft
approved
ordered
partially_received
received
cancelled
```

Document transition rules and reject invalid transitions on the backend.

---

## Date & Money Rules

### Money

- Use DECIMAL in MySQL.
- Never use floating-point columns for money.
- Store currency explicitly where multi-currency is possible.
- Define rounding rules centrally.

### Dates

- Store timestamps consistently.
- Use application timezone configuration.
- Agreement date overlap logic must use explicit inclusive/exclusive boundary rules.
- Do not infer "active" only from status if dates also matter; define the canonical rule.

---

## AI Capability Rules

AI features may assist users but must not bypass permissions.

AI must:

- operate only on data the authenticated user can access;
- respect current branch context;
- cite or identify underlying ERP records where practical;
- distinguish generated interpretation from stored system-of-record data;
- avoid executing financial postings or destructive changes without explicit user confirmation and backend authorization;
- log sensitive AI-triggered actions.

See [AI Guidelines](docs/ai/ai-guidelines.md).

---

## Required Tests for Sensitive Changes

Any change affecting these areas requires automated tests:

- branch scoping;
- user authorization;
- owner agreements;
- tenant agreements;
- availability/occupancy;
- payments;
- receipts;
- inventory movement;
- work-order consumption;
- purchase receiving;
- branch switching;
- super-admin access.

See [Testing Strategy](docs/testing.md).

---

## Change Workflow for Coding Agents

Before editing:

1. read this file;
2. read the relevant file under `docs/`;
3. inspect existing models/migrations/API contracts;
4. identify branch scope and authorization implications;
5. identify financial, stock, and occupancy invariants.

While editing:

1. make the smallest coherent change;
2. avoid unrelated refactors;
3. preserve existing API behavior unless change is approved;
4. add/update tests;
5. update documentation when a domain rule changes.

Before completing:

1. run relevant backend tests;
2. run relevant frontend tests;
3. run linters/formatters;
4. verify no cross-branch leakage;
5. verify migrations are reversible where practical;
6. verify no secrets or credentials were added.

---

## Documentation Index

- [Architecture](docs/architecture.md)
- [Domain Model](docs/domain/domain-model.md)
- [Property & Leasing](docs/domain/property-and-leasing.md)
- [Payments & Accounting](docs/domain/payments-accounting.md)
- [Maintenance & Inventory](docs/domain/maintenance-inventory.md)
- [Backend Guidelines](docs/backend/backend-guidelines.md)
- [API Conventions](docs/backend/api-conventions.md)
- [API Schema Library](docs/api-schema-library.md)
- [Database Guidelines](docs/backend/database-guidelines.md)
- [Frontend Guidelines](docs/frontend/frontend-guidelines.md)
- [AI Guidelines](docs/ai/ai-guidelines.md)
- [Security](docs/security.md)
- [Testing](docs/testing.md)
- [Development Workflow](docs/development-workflow.md)
- [Glossary](docs/glossary.md)
