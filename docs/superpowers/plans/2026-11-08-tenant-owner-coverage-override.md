# Tenant Owner-Coverage Override Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Allow a specifically authorized user to create a tenant agreement outside its source owner-agreement dates with a mandatory reason, visible warning, and immutable audit trail.

**Architecture:** Keep the existing owner-coverage check as the default. Add an explicit override option only to tenant creation, authorize it with a new branch-scoped permission, and pass the option into the shared availability service so property linkage, branch scope, owner status, and tenant occupancy checks remain enforced. Return the warning through Laravel resource `additional()` data and record the override after the agreement is saved.

**Tech Stack:** Laravel/PHP, MySQL, PHPUnit feature tests, Angular/TypeScript reactive form.

**Spec:** `docs/superpowers/specs/2026-11-08-tenant-owner-coverage-override-design.md`

## Global Constraints

- Normal tenant creation must continue rejecting dates outside source owner coverage.
- The override must not bypass branch authorization, source property coverage, owner status validation, or overlapping tenant occupancy checks.
- Tenant PATCH, approval, extension, and renewal remain unchanged.
- The override reason is required and must be stored in the audit metadata.
- No financial records or payment schedules are silently changed.
- Update `docs/api-schema-library.md` for the changed create contract.

## Review Focus

- Override requested without permission: return `403` and create no agreement; test in the backend feature test.
- Override requested without a reason: return validation failure and create no agreement; test in the backend feature test.
- Authorized override with overlapping tenant dates: still return `PROPERTY_NOT_AVAILABLE`; test in the backend feature test.
- Authorized override with invalid source property/owner relationship: still return coverage failure; test in the backend feature test.
- Existing normal create path: preserve the current coverage rejection and response shape; test in the backend feature test.

---

### Task 1: Backend permission and request contract

**Files:**
- Modify: `backend/database/seeders/RoleSeeder.php`
- Modify: `backend/database/migrations/2026_10_02_000002_add_management_permissions.php`
- Create: `backend/database/migrations/2026_11_08_000001_add_agreement_coverage_override_permission.php`
- Modify: `backend/app/Http/Requests/Api/V1/Agreements/StoreTenantAgreementRequest.php`
- Test: `backend/tests/Feature/Api/Agreements/TenantPropertyAvailabilityTest.php`

**Interfaces:**
- Request fields: `coverage_override` boolean, `coverage_override_reason` nullable string with max length 2000.
- Permission key: `agreements.override_coverage`.
- Validation must require a non-blank reason whenever `coverage_override` is true.

- [ ] **Step 1: Write the failing tests**

Add focused feature tests asserting:

1. A tenant date range outside owner coverage without `coverage_override` returns `422` with `OWNER_AGREEMENT_DATE_COVERAGE_REQUIRED`.
2. An override without `coverage_override_reason` returns `422` with validation errors.
3. The permission exists after migration/seed setup.

- [ ] **Step 2: Run the focused tests and verify the expected failures**

Run from `backend`:

```powershell
php artisan test tests/Feature/Api/Agreements/TenantPropertyAvailabilityTest.php --filter="coverage|override"
```

Expected: the new override-specific assertions fail because the request contract and permission do not yet exist.

- [ ] **Step 3: Implement the permission and validation**

Add the permission to the existing permission bootstrap and a reversible migration that inserts it and grants it to the current system roles, following the existing management-permission migration pattern. Add the two request fields and conditional reason validation; default the boolean to false in validated data.

- [ ] **Step 4: Run the focused tests and verify validation passes**

Run the same command. Expected: request-level tests pass; end-to-end authorized override tests remain red until Task 2.

### Task 2: Backend override behavior, warning, and audit

**Files:**
- Modify: `backend/app/Services/PropertyAvailabilityService.php`
- Modify: `backend/app/Http/Controllers/Api/V1/TenantAgreementController.php`
- Test: `backend/tests/Feature/Api/Agreements/TenantPropertyAvailabilityTest.php`

**Interfaces:**
- `PropertyAvailabilityService::assertOwnerCoverage(..., bool $allowOutsideCoverage = false): void`.
- Successful override response adds:

```json
{
  "warnings": [
    {
      "code": "OWNER_AGREEMENT_COVERAGE_OVERRIDDEN",
      "message": "Tenant dates exceed the source owner agreement coverage by authorization."
    }
  ]
}
```

- Audit action: `tenant_agreement.coverage_overridden`; metadata includes `reason`, `source_owner_agreement_ids`, `tenant_start_date`, `tenant_end_date`, and each source owner's `start_date`/`end_date`.

- [ ] **Step 1: Write the failing behavior tests**

Add tests asserting:

1. An authorized override creates the tenant agreement and returns the warning.
2. The audit log contains the dedicated action, actor, branch, reason, tenant dates, and owner coverage dates.
3. A user without `agreements.override_coverage` receives `403` and no tenant agreement is persisted.
4. Authorized override does not bypass a blocking tenant overlap and does not bypass a missing property link.

- [ ] **Step 2: Run the focused tests and verify they fail for the intended reasons**

Run the focused test command from Task 1. Expected: authorized override is rejected by the current coverage guard; permission-denied and invariant tests establish the expected security boundary.

- [ ] **Step 3: Implement the minimal backend behavior**

Before the create transaction, when `coverage_override` is true, check `$request->user()->hasPermission('agreements.override_coverage', $branchContext->id())`; reject unauthorized requests with `403`. Pass the validated flag into `assertOwnerCoverage`. Skip only the date comparison when allowed. Keep `assertAvailable` unchanged. After save, record the dedicated audit event with the reason and coverage details. Return `TenantAgreementResource(...)->additional(['warnings' => ...])` only when the override was used; leave normal responses unchanged.

- [ ] **Step 4: Run the focused tests and verify they pass**

Run:

```powershell
php artisan test tests/Feature/Api/Agreements/TenantPropertyAvailabilityTest.php
```

Expected: all tests in the file pass.

### Task 3: Angular creation warning and confirmation

**Files:**
- Modify: `frontend/src/app/features/tenant-agreements/tenant-agreement-form.component.ts`
- Modify: `frontend/src/app/core/api/tenant-agreements-api.service.ts` only if its payload type needs the two fields.
- Test: the existing tenant-agreement form/component test location, or create the smallest focused spec beside the component if none exists.

**Interfaces:**
- Creation payload sends `coverage_override: true` and `coverage_override_reason` only after confirmation.
- The form displays the owner/tenant date mismatch and requires a non-empty reason before resubmitting.
- The existing API error handler displays `403`/validation responses without treating them as successful creation.

- [ ] **Step 1: Write the failing component test**

Assert that when tenant end date exceeds the selected owner end date, submission pauses for confirmation, does not call the API before confirmation, and sends the override flag plus reason after confirmation.

- [ ] **Step 2: Run the component test and verify it fails**

Run the exact frontend test command from `frontend/package.json` for the focused spec. Expected: the current form submits immediately without override fields.

- [ ] **Step 3: Implement the smallest UI flow**

Compare the selected owner agreement dates with the form dates during create. On mismatch, show a warning confirmation UI with a required reason; on confirmation, submit the two fields. Do not add an override control for edit mode. Preserve the normal form path when dates are within coverage.

- [ ] **Step 4: Run the focused component test and frontend checks**

Run the focused test, then the project test/lint/build commands from `frontend/package.json`. Expected: all pass.

### Task 4: Contract documentation and full verification

**Files:**
- Modify: `docs/api-schema-library.md`
- Modify: `docs/user-guide.md` if the existing tenant creation instructions need the override warning flow.

- [ ] **Step 1: Document the optional create fields, permission, warning, and error behavior**

Document that the normal coverage rule remains default; authorized creation may include the two override fields; missing permission is `403`; missing reason is validation error; successful override returns warning code `OWNER_AGREEMENT_COVERAGE_OVERRIDDEN`.

- [ ] **Step 2: Run the complete backend and frontend verification**

Run the exact commands in both submodules' `composer.json` and `package.json`, including formatter/linter/build where configured. Expected: no new failures.

- [ ] **Step 3: Inspect the final diff**

Confirm no changes touch tenant PATCH/lifecycle behavior, branch scoping, occupancy overlap checks, posted financial records, or secrets.

- [ ] **Step 4: Commit the completed change**

Use a focused commit message such as:

```text
feat: authorize tenant owner-coverage overrides
```
