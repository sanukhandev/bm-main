# Architecture

## 1. System Overview

Baithul Madeena is a multi-branch real-estate ERP.

```text
Angular SPA
    ↓ HTTPS / JSON
Laravel REST API
    ↓
MySQL
```

The repository is intended to be organized as:

```text
baithul-madeena/
├── AGENTS.md
├── docs/
├── backend/       # Git submodule
└── frontend/      # Git submodule
```

The main repository owns cross-project documentation, integration conventions, environment orchestration, and release coordination.

---

## 2. Architectural Priorities

In priority order:

1. authorization and branch isolation;
2. financial correctness;
3. tenancy/occupancy correctness;
4. inventory traceability;
5. auditability;
6. API stability;
7. maintainability;
8. performance;
9. AI usability.

---

## 3. Branch Isolation

Branch isolation is a core security boundary.

### Data ownership

Most operational records should be associated with a branch directly or through an immutable parent relation.

Examples of directly branch-owned records may include:

- properties;
- agreements;
- payments;
- receipts;
- work orders;
- inventory locations;
- purchase orders;
- invoices;
- dashboard aggregates.

Master/reference data may be global where explicitly intended.

### Context resolution

Recommended request flow:

```text
authentication
  ↓
user identity
  ↓
authorized branch list
  ↓
active branch resolution
  ↓
branch-aware authorization
  ↓
branch-scoped repository/query
```

### Normal users

For normal users, all branch-owned queries must be restricted to authorized active branch context.

### Super admins

A super admin can:

- choose one branch and behave in that branch context; or
- request explicitly supported all-branch reports.

Do not accidentally make every endpoint unscoped for super admins. Cross-branch behavior should be deliberate.

### Branch switching

Branch switching should:

1. verify requested branch exists;
2. verify the user may access it, unless the user is super admin;
3. persist active branch context in a secure mechanism;
4. invalidate branch-dependent frontend caches;
5. return enough branch metadata for the UI;
6. create an audit event for privileged switching when required.

---

## 4. Suggested Laravel Layering

```text
HTTP
├── Controllers
├── Form Requests
├── Resources
└── Middleware

Application
├── Actions
├── Services
├── DTOs
└── Queries

Domain
├── Models
├── Enums
├── Policies
├── Value Objects
└── Domain Rules

Infrastructure
├── Persistence
├── Integrations
├── AI Providers
├── File Storage
└── Notifications
```

Do not force a theoretical architecture onto a mature codebase. Follow existing project patterns if they already provide the same separation of concerns.

---

## 5. Suggested Angular Layering

```text
src/app/
├── core/
│   ├── auth/
│   ├── branch-context/
│   ├── guards/
│   ├── interceptors/
│   └── services/
├── shared/
│   ├── components/
│   ├── directives/
│   ├── pipes/
│   ├── models/
│   └── utilities/
├── features/
│   ├── dashboard/
│   ├── customers/
│   ├── properties/
│   ├── owner-agreements/
│   ├── tenant-agreements/
│   ├── payments/
│   ├── maintenance/
│   ├── inventory/
│   ├── customers/        # includes owner, tenant and vendor roles
│   ├── purchase-orders/
│   └── invoices/
└── app.routes.ts
```

Prefer feature boundaries over technical dumping grounds.

---

## 6. Transaction Boundaries

Database transactions should be used when a business operation updates multiple dependent records.

Examples:

- activating an agreement and creating its installment schedule;
- posting a payment and issuing a receipt;
- receiving a purchase order and increasing inventory;
- completing inventory consumption for a work order;
- returning consumed items to inventory;
- scrapping stock and recording a write-off.

A partial success in these flows is unacceptable.

---

## 7. Events and Side Effects

Where useful, publish domain/application events after a successful transaction.

Examples:

```text
OwnerAgreementActivated
TenantAgreementActivated
PaymentPosted
CashReceiptIssued
PurchaseReceiptIssued
PurchaseOrderReceived
InventoryConsumed
InventoryReturned
StockScrapped
WorkOrderCompleted
```

Side effects such as notifications, PDF generation, analytics, or AI indexing should not corrupt the core transaction if they fail.

---

## Operational dashboard read model

The operational dashboard is served by `GET /api/v1/dashboard/operational`.
Its counts and attention lists are calculated by the backend for the verified
active branch. Operational access does not grant Accounts access: receivables,
payables, overdue installment amounts, and pending cheque values are omitted
unless the user has `accounts.view`.

## 8. Auditability

Audit at least:

- logins/security events where applicable;
- branch changes for privileged users;
- customer changes;
- property ownership/structure changes;
- agreement status changes;
- payment posting;
- receipt generation/voiding;
- stock adjustments;
- stock scrap;
- work-order completion;
- purchase order approval/receipt;
- sensitive AI actions.

Useful audit fields:

```text
actor_user_id
branch_id
action
entity_type
entity_id
before_json
after_json
ip_address
user_agent
created_at
```

Do not store secrets in audit payloads.

---

## 9. File and Attachment Storage

Business entities may later require attachments such as:

- Emirates ID/passport/company documents;
- owner documents;
- property documents;
- tenancy contracts;
- cheques;
- invoices;
- work-order photos.

File authorization must follow entity authorization.

Never expose storage paths directly if they bypass access control.

---

## 10. Performance

Common optimization targets:

- dashboard counts;
- agreement lists;
- property availability;
- payment schedules;
- inventory balances;
- work-order history.

Use:

- indexes;
- eager loading;
- pagination;
- aggregate queries;
- caching only where invalidation is reliable.

Branch ID should typically participate in indexes for large branch-scoped tables.

---

## 11. Integration Boundary

External integrations should be isolated behind interfaces/adapters.

Potential examples:

- payment gateways;
- bank import;
- SMS/WhatsApp;
- email;
- AI model providers;
- accounting integrations;
- document signing.

Do not put provider-specific behavior throughout domain logic.

## Central audit trail

Meaningful successful mutations write an append-only `audit_logs` record through
the backend `AuditService`. Records carry the actor, active/entity branch,
stable entity alias, action, compact before/after snapshots, safe metadata, and
request metadata. Domain histories such as agreement status history remain
authoritative for their own timelines; the central trail provides the
cross-domain actor/action view. Audit viewing is branch-scoped through
`GET /api/v1/audit-logs` and requires `audit.view`. There is no all-branches
audit mode in Phase 1.
