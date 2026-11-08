# Agreement Print Template Design

## Intent

Make owner and tenant agreement detail pages produce a print-ready HTML agreement that follows the supplied one-page PDF references while showing all data currently available from the API. The browser print dialog opens from the existing Print actions so users can save or print a PDF without a new backend document service.

## Goals

- Use one shared Angular print template with owner/tenant-specific labels and terms.
- Show agreement number, status, creation date, active branch name, party identity/contact details, properties, utilities/amenities, lease dates, payment terms, installments, additional payments, notes, and signatures when available.
- Use the reference PDF wording for tenant and owner terms for now, kept in code for later configuration.
- Preserve A4 print styling, readable page breaks, and hide the normal application chrome during printing.
- Keep the business party name as `BAITHUL MADEENA REAL ESTATE` for tenant agreements and as the tenant party on owner agreements.

## Non-goals

- No server-side PDF generation or new API endpoint.
- No new persisted agreement, property, or amenity fields.
- No configurable terms editor in this change.
- No change to agreement lifecycle, payment, or authorization behavior.

## Constraints and data decisions

- Use active branch data from `BranchContextService`; fall back to the agreement branch ID when a branch name is unavailable.
- Render missing values as an em dash or omit optional sections rather than inventing values.
- Derive utilities/amenities from the linked property fields (`utility_details`, electricity/cooling/gas fields, and notes) and show all linked properties.
- Map payment terms from agreement totals, count/frequency/mode, installments, additional payments, and notes/terms text.
- Keep reference terms split by document type: four tenant clauses and five owner clauses, with bilingual display where the PDF text is available.
- `window.print()` remains the browser print mechanism; the print sheet is rendered before opening the dialog.

## Approaches considered

1. Extend the existing shared `BmAgreementPrintSheetComponent` (chosen): smallest change, one print CSS/layout, and a single future location for configurable terms.
2. Separate owner and tenant print components: more independent layouts but duplicates data mapping and print CSS.
3. Add a new print route/window: isolates printing but adds routing and duplicate agreement loading for no functional gain.

## Files in scope

- `frontend/src/app/shared/components/bm-agreement-print-sheet/bm-agreement-print-sheet.component.ts`: shared HTML template, typed print inputs, terms, property/utility/payment rendering, and A4 print CSS.
- `frontend/src/app/features/owner-agreements/owner-agreement-detail.component.ts`: pass owner agreement, owner party, branch, and print data; open the print dialog after render.
- `frontend/src/app/features/tenant-agreements/tenant-agreement-detail.component.ts`: pass tenant agreement, business/tenant parties, branch, and print data; open the print dialog after render.
- `frontend/src/app/shared/models/agreement.models.ts` or existing property models only if a small type adjustment is required for the already returned data.
- Relevant frontend specs: add focused assertions for owner/tenant labels, branch/party names, available utility/payment data, terms selection, and print invocation.

## Acceptance checks

- Owner and tenant Print actions open the browser print dialog and print only the agreement sheet.
- Printed output contains branch name, party names, agreement dates, payment terms, linked properties, available utility/amenity details, and available notes/terms.
- Tenant output identifies `BAITHUL MADEENA REAL ESTATE` as lessor; owner output identifies it as tenant.
- Owner output uses the owner reference clauses; tenant output uses the tenant reference clauses.
- Missing optional data does not produce broken layout or runtime errors.
- Angular focused tests, full frontend tests, production build, and `git diff --check` pass.
