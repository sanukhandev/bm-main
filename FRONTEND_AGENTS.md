# Frontend AGENTS.md Template

Use this file as `frontend/AGENTS.md` inside the Angular submodule.

## Scope

This submodule implements the Baithul Madeena Angular frontend.

Read the parent repository's `AGENTS.md` and `docs/frontend/*` before making changes.

## Non-Negotiable Rules

- Treat active branch as first-class application context.
- Clear/refetch branch-sensitive data after branch switching.
- Never rely on frontend authorization as a security boundary.
- Use typed TypeScript models.
- Centralize API access.
- Prefer reactive forms for ERP workflows.
- Prevent accidental duplicate financial submissions.
- Show loading, empty, error, and permission states.
- Never compute authoritative financial or dashboard totals from partial client-side datasets.

## UI Areas

```text
Dashboard
Customers
Properties
Owner Agreements
Tenant Agreements
Payments / Receipts
Maintenance
Inventory
Vendors
Purchase Orders
Invoices
Administration
AI Assistant
```

## Coding Workflow

Before completion run the project's:

```text
formatter
linter
unit/component tests
build
```

Inspect `package.json` for the exact commands.
