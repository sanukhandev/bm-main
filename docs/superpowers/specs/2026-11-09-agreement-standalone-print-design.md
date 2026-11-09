# Agreement standalone print window

## Intent

Make owner and tenant agreement printing match the supplied standalone HTML sample: an A4 document with bilingual headings, bordered data tables, payment terms, property/services, agreement clauses, and signature areas. Printing must open in a new browser window and trigger the browser print dialog automatically.

## Goals

- Use the agreement, party, branch, property, utility, installment, and additional-payment data already loaded by the detail page.
- Keep owner and tenant terms distinct and preserve the current fixed terms until terms become configurable.
- Render a print-only document with the sample's structure: letterhead space, centered agreement title, party tables, leased-premise table, utilities/services, lease period, payment terms, bilingual clauses, and signatures.
- Open the document in a new window and call `window.print()` after it is loaded.
- Keep the existing API contracts and detail-page data loading unchanged.

## Non-goals

- No new backend endpoint or PDF generation service.
- No database/schema changes.
- No external CDN dependency for print behavior.
- No configurable clause editor in this change.

## Approaches

1. Render a standalone HTML document from a second data-mapping implementation. This provides complete independence but duplicates the agreement template and risks data drift.
2. Recommended: keep `BmAgreementPrintSheetComponent` as the single rendered data source, redesign its template/CSS to the sample layout, and expose a method that clones its rendered HTML into a new print window. This minimizes duplication and preserves all currently available fields.
3. Add a dedicated Angular print route that reloads agreement data. This is reusable but adds routing, authentication, branch-context, and a second loading path for a document that is already loaded.

## Design

The shared print component will expose `openPrintWindow()`. It will clone the fully rendered `.agreement-print-sheet`, build a complete HTML document with A4 print CSS, copy the component's required styles, close the new document, and schedule `printWindow.print()` after load. A blocked popup will be handled without changing agreement state.

The detail components will call the shared component through `ViewChild`; their current print buttons and agreement API calls remain unchanged. Tests will cover the generated document shell/print behavior and the existing data fallbacks. Verification will include an Angular build and component tests.

## Builder decisions

- Use the current business name and branch metadata already passed to the component.
- Use API data when present and display an em dash for missing optional values.
- Keep the sample's bilingual labels and current owner/tenant clause arrays.
- Use A4 dimensions, zero page margins through `@page`, thin borders, compact typography, and `break-inside` rules to keep sections print-friendly.
