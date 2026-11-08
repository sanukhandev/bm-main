# Agreement PDF Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Upgrade the existing owner and tenant agreement print sheet into a complete, bilingual, branch-aware HTML agreement that opens the browser print dialog and matches the supplied PDF references.

**Architecture:** Keep one shared `BmAgreementPrintSheetComponent`. The detail pages provide the agreement, active branch, party/customer data, and linked properties; the component owns the print layout, utility/payment normalization, and owner-versus-tenant clause sets. Printing remains client-side through `window.print()` and existing `@media print` rules.

**Tech Stack:** Angular standalone components, TypeScript, existing Angular test runner, browser print CSS, existing agreement/property/customer/branch models.

**Spec:** `docs/superpowers/specs/2026-11-08-agreement-pdf-design.md`

## Global Constraints

- No server-side PDF generation or new API endpoint.
- No new persisted agreement, property, or amenity fields.
- Keep reference terms hard-coded for now and separate by owner/tenant agreement type.
- Render missing optional values as an em dash or omit optional sections.
- Use the active branch from `BranchContextService` and preserve the business name `BAITHUL MADEENA REAL ESTATE`.
- Keep A4 print output readable, with no application chrome in print.

## Review Focus

- Missing customer/property/branch data must render safely without runtime errors or broken labels.
- Multiple linked properties must all appear, including each property's utility/amenity details.
- Installment and additional-payment rows must preserve amount, due date, mode, status, and particulars when present.
- Owner and tenant documents must select different PDF-derived terms and party labels.
- Print invocation must occur after the print sheet is rendered and must not print the application screen.

---

### Task 1: Add print-sheet data contracts and reference content

**Files:**
- Modify: `frontend/src/app/shared/components/bm-agreement-print-sheet/bm-agreement-print-sheet.component.ts`
- Test: `frontend/src/app/shared/components/bm-agreement-print-sheet/bm-agreement-print-sheet.component.spec.ts`

**Interfaces:**
- Consumes: `OwnerAgreement | TenantAgreement`, `Branch | null`, existing `Customer` and `Property` shapes.
- Produces: `PrintableParty`, `PrintableBranch`, and component inputs for branch, party details, and linked properties; the rendered HTML remains the component's public contract.

- [ ] **Step 1: Write failing component tests**

  Add focused TestBed tests that set representative owner and tenant inputs and assert:

  - branch name, agreement number, date range, and status render;
  - tenant output contains `BAITHUL MADEENA REAL ESTATE` as lessor and the tenant name;
  - owner output contains the owner name and `BAITHUL MADEENA REAL ESTATE` as tenant;
  - customer contact/identity/address fields, property rows, utility/amenity values, payment terms, installments, additional payments, and notes render when available;
  - owner-only and tenant-only reference terms are selected correctly;
  - empty optional inputs render an em dash or omit the section without throwing.

- [ ] **Step 2: Run the focused test to verify it fails**

  Run from `frontend`: `npx ng test --no-watch --no-progress --include='src/app/shared/components/bm-agreement-print-sheet/bm-agreement-print-sheet.component.spec.ts'`

  Expected: FAIL because the new inputs and complete sections do not yet exist.

- [ ] **Step 3: Implement the shared contracts and rendering**

  Extend the component with typed party/branch inputs and the minimum helpers needed to normalize nested property resources, format money/status/payment mode, flatten property utilities, and display all available fields. Replace the generic clauses with the four tenant clauses and five owner clauses from the supplied PDFs, keeping bilingual text where available. Keep the reference spelling/meaning intact and do not add configurable-term behavior.

- [ ] **Step 4: Run the focused test to verify it passes**

  Expected: PASS for all print-sheet tests.

- [ ] **Step 5: Commit**

  `git add frontend/src/app/shared/components/bm-agreement-print-sheet/bm-agreement-print-sheet.component.ts frontend/src/app/shared/components/bm-agreement-print-sheet/bm-agreement-print-sheet.component.spec.ts && git commit -m "feat: expand agreement print sheet"`

### Task 2: Wire owner and tenant detail pages to branch-aware printing

**Files:**
- Modify: `frontend/src/app/features/owner-agreements/owner-agreement-detail.component.ts`
- Modify: `frontend/src/app/features/tenant-agreements/tenant-agreement-detail.component.ts`
- Modify: `frontend/src/app/shared/components/bm-agreement-print-sheet/bm-agreement-print-sheet.component.ts` only if the caller contract from Task 1 needs a small integration adjustment.
- Test: extend the relevant detail component tests only if existing test coverage is present; otherwise rely on the shared component contract tests and build verification.

**Interfaces:**
- Consumes: Task 1 print-sheet inputs and `BranchContextService.activeBranch()`.
- Produces: owner/tenant detail pages that pass real agreement/customer/property/branch data and trigger browser printing after the sheet is present.

- [ ] **Step 1: Add the branch-context dependency and caller mappings**

  Inject `BranchContextService`, pass the active branch, map owner/tenant customer details into the correct first/second party slots, and preserve the exact business name `BAITHUL MADEENA REAL ESTATE` in the opposite party slot required by each agreement type.

- [ ] **Step 2: Update the print action**

  Keep the existing Print and Print Sheet actions, but schedule `window.print()` after the current Angular view has rendered (`setTimeout(..., 0)`) so the browser print dialog opens with the complete HTML sheet. Do not introduce a popup window or server request.

- [ ] **Step 3: Run focused frontend tests and build**

  Run the print-sheet spec, the existing tenant-agreement form spec, the full frontend test suite, and `npm run build` from `frontend`.

  Expected: all tests pass and the production build succeeds.

- [ ] **Step 4: Commit**

  `git add frontend/src/app/features/owner-agreements/owner-agreement-detail.component.ts frontend/src/app/features/tenant-agreements/tenant-agreement-detail.component.ts && git commit -m "feat: wire branch-aware agreement printing"`

### Task 3: Verify print output and handoff

**Files:**
- Modify: only files required by verification findings.

**Interfaces:**
- Consumes: completed shared print component and detail-page wiring.
- Produces: a verified `feat/agreement-pdf` branch with no API or submodule changes.

- [ ] **Step 1: Run static checks**

  Run `git diff --check` in the parent worktree and each modified submodule, then run the frontend formatter/linter command already defined by the repository if available.

- [ ] **Step 2: Inspect browser print rendering**

  Open representative owner and tenant detail pages, invoke Print, confirm the print preview contains only the agreement sheet, has A4 pagination, no clipped tables, and includes branch/party/property/payment/utility/terms data.

- [ ] **Step 3: Review the final diff**

  Confirm no reference PDFs, generated artifacts, secrets, or unrelated refactors were added; confirm the parent gitlink still points at the existing backend/frontend commits.

- [ ] **Step 4: Commit any verification-only fixes**

  Use a focused commit only if the visual or static checks identify a real issue.
