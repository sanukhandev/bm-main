# Proposed Project Structure

```text
baithul-madeena/
├── AGENTS.md
├── PROJECT_STRUCTURE.md
├── .gitmodules
├── docs/
│   ├── README.md
│   ├── architecture.md
│   ├── security.md
│   ├── testing.md
│   ├── development-workflow.md
│   ├── glossary.md
│   ├── backend/
│   │   ├── backend-guidelines.md
│   │   ├── api-conventions.md
│   │   └── database-guidelines.md
│   ├── frontend/
│   │   └── frontend-guidelines.md
│   ├── domain/
│   │   ├── domain-model.md
│   │   ├── property-and-leasing.md
│   │   ├── payments-accounting.md
│   │   └── maintenance-inventory.md
│   └── ai/
│       └── ai-guidelines.md
├── backend/               # Git submodule: Laravel API
└── frontend/              # Git submodule: Angular application
```

## Suggested backend submodule structure

```text
backend/
├── AGENTS.md
├── app/
│   ├── Actions/
│   ├── DTOs/
│   ├── Enums/
│   ├── Exceptions/
│   ├── Http/
│   │   ├── Controllers/
│   │   ├── Middleware/
│   │   ├── Requests/
│   │   └── Resources/
│   ├── Models/
│   ├── Policies/
│   ├── Services/
│   └── Support/
├── database/
│   ├── factories/
│   ├── migrations/
│   └── seeders/
├── routes/
│   └── api.php
└── tests/
    ├── Feature/
    └── Unit/
```

## Suggested frontend submodule structure

```text
frontend/
├── AGENTS.md
├── src/
│   └── app/
│       ├── core/
│       │   ├── auth/
│       │   ├── branch-context/
│       │   ├── guards/
│       │   ├── interceptors/
│       │   └── services/
│       ├── shared/
│       └── features/
│           ├── dashboard/
│           ├── customers/
│           ├── properties/
│           ├── owner-agreements/
│           ├── tenant-agreements/
│           ├── payments/
│           ├── maintenance/
│           ├── inventory/
│           ├── vendors/
│           ├── purchase-orders/
│           └── invoices/
└── ...
```

The exact folder layout should follow established conventions in each submodule once implementation exists.
