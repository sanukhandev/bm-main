# Agreement standalone print window Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace in-page agreement printing with a sample-shaped standalone A4 HTML document that opens in a new window and auto-triggers printing.

**Architecture:** Keep `BmAgreementPrintSheetComponent` as the single data-rendering source. Redesign its template/styles to match the supplied bilingual table layout, then expose `openPrintWindow()` to clone the rendered document into a new window. Owner and tenant detail components delegate their existing print buttons to that method.

**Tech Stack:** Angular standalone components, TypeScript, browser `window.open()`/`window.print()`, Vitest via Angular test runner.

**Spec:** `docs/superpowers/specs/2026-11-09-agreement-standalone-print-design.md`

## Global Constraints

- Preserve existing owner/tenant API contracts and loaded data flow.
- Use API values when available and `—` for missing optional values.
- Preserve the existing fixed bilingual owner and tenant terms.
- No backend, database, dependency, or configurable-terms changes.
- Keep print behavior usable when the browser blocks a popup.

## Review Focus

- Popup blocked or `window.open()` returns `null`: print action must fail safely without changing agreement state; test in Task 2.
- Optional party, branch, property, utility, payment, and notes fields: the standalone document must retain existing fallbacks; test in Task 2.
- Tenant and owner clauses: the correct bilingual clause set must remain selected; test in Task 2.
- Print timing: the new document must print only after its HTML is loaded; test in Task 2.
- Detail-page delegation: both owner and tenant buttons must call the shared print-window method; test in Task 3.

### Task 1: Add failing print-window component tests

**Files:**
- Modify: `frontend/src/app/shared/components/bm-agreement-print-sheet/bm-agreement-print-sheet.component.spec.ts`

**Interfaces:**
- Consumes: the existing component fixture and agreement fixtures.
- Produces: assertions for `openPrintWindow()` and the sample-shaped standalone document.

- [ ] **Step 1: Write the failing tests**

  Add a test that stubs `window.open()` with a fake window/document, calls `openPrintWindow()`, and asserts the generated document contains the A4 shell, bilingual title, agreement number, payment terms, property/service data, terms, and signature labels. Add a test that `window.open()` returning `null` does not throw or call print.

- [ ] **Step 2: Run the focused test and verify it fails for the missing behavior**

  Run from `frontend`:

  ```bash
  npx ng test --no-watch --no-progress --include=src/app/shared/components/bm-agreement-print-sheet/bm-agreement-print-sheet.component.spec.ts
  ```

  Expected: failure because `openPrintWindow()` and the standalone document behavior do not exist.

### Task 2: Implement the sample-shaped standalone print document

**Files:**
- Modify: `frontend/src/app/shared/components/bm-agreement-print-sheet/bm-agreement-print-sheet.component.ts`
- Test: `frontend/src/app/shared/components/bm-agreement-print-sheet/bm-agreement-print-sheet.component.spec.ts`

**Interfaces:**
- Consumes: current `agreement`, party, branch, and property inputs plus existing computed data helpers.
- Produces: `openPrintWindow(): void` and the rendered `.agreement-print-sheet` document used by both detail pages.

- [ ] **Step 1: Implement `openPrintWindow(): void` minimally**

  Use the component host’s rendered `.agreement-print-sheet` HTML as the source, create a new same-origin window, write a complete HTML document with UTF-8 metadata, A4 `@page` rules, the sample’s compact table layout, and the component’s print styles, then close the document and call `printWindow.print()` after load. Return safely when the popup is blocked.

- [ ] **Step 2: Redesign the component template and styles**

  Shape the markup as the supplied sample: reserved letterhead area, centered bilingual agreement title, owner/lessor and tenant tables, leased-premise table, electricity/water/sewage and furniture/service rows, lease period, payment terms, payment schedule, bilingual numbered terms, and signature/footer tables. Keep current API-backed values and owner/tenant clause arrays.

- [ ] **Step 3: Run the focused tests and verify they pass**

  Run the same focused Angular test command. Expected: all component tests pass, including the popup-blocked case and existing data fallback checks.

### Task 3: Delegate owner and tenant print actions

**Files:**
- Modify: `frontend/src/app/features/owner-agreements/owner-agreement-detail.component.ts`
- Modify: `frontend/src/app/features/tenant-agreements/tenant-agreement-detail.component.ts`
- Modify: `frontend/src/app/features/owner-agreements/owner-agreement-detail.component.spec.ts`
- Modify: `frontend/src/app/features/tenant-agreements/tenant-agreement-detail.component.spec.ts`

**Interfaces:**
- Consumes: `BmAgreementPrintSheetComponent.openPrintWindow()` from Task 2.
- Produces: both existing print buttons opening the standalone print document.

- [ ] **Step 1: Write the failing delegation tests**

  Replace the current `window.print()` timer assertions with a stubbed print-sheet object and assert each detail component’s `printAgreement()` calls `openPrintWindow()` exactly once.

- [ ] **Step 2: Run the two focused detail tests and verify they fail**

  Run the tenant and owner detail spec includes. Expected: failure because both methods still call the page-level `window.print()` timer.

- [ ] **Step 3: Wire `@ViewChild(BmAgreementPrintSheetComponent)` and delegate**

  Add a view-child reference in both detail components and change `printAgreement()` to call the shared component method when available. Keep the existing template component instances and buttons unchanged.

- [ ] **Step 4: Run the focused detail tests and verify they pass**

  Run the same two focused spec commands. Expected: both delegation tests pass.

### Task 4: Full verification and handoff

**Files:**
- Verify: all frontend changes above and the design/plan documents.

- [ ] **Step 1: Run formatting/diff checks**

  Run `git diff --check` in the parent and frontend submodule.

- [ ] **Step 2: Run the complete frontend build**

  Run `npm run build` from `frontend`; expected Angular bundle generation succeeds.

- [ ] **Step 3: Run the complete frontend test command**

  Run `npx ng test --no-watch --no-progress`; report any existing runner/worker failures separately from feature failures.

- [ ] **Step 4: Review the final diff**

  Confirm only the print component, two detail delegates, their tests, and the approved design/plan docs changed; confirm no API or database files changed.

- [ ] **Step 5: Commit the implementation**

  Commit the frontend submodule changes with `feat: open agreements in standalone print window`, then update and commit the parent submodule pointer if the workflow requires it.
