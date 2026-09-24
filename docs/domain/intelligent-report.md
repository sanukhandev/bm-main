# Intelligent Report

The Intelligent Report is a management read model for an authorized branch or
an explicitly selected Overall Business scope for Super Admin users.

## Definitions

- **Cash Position** is posted inward minus posted outward account transactions
  in the selected period. Void transactions are excluded by the existing
  financial rules.
- **Operational Profit/Loss** is an ERP-captured management metric. It uses
  captured inward transactions as operating income and captured outward
  transactions as operating costs, with maintenance and petty-cash source
  classifications shown separately where available. It is not a statutory or
  general-ledger Profit & Loss statement.
- Receivables, payables, collection efficiency, occupancy, cheque values, and
  expiry counts reuse the existing branch-aware domain data.

## Security and scope

The API requires `accounts.view`. A normal user receives only the verified
active branch. Super Admin may request `scope=overall` explicitly; this does
not change the branch behavior of other ERP endpoints. Client-supplied branch
IDs are not used to establish scope.

## Findings

Leakage findings are deterministic backend conditions, currently including
overdue receivables, bounced-cheque exposure, and expiring agreements with
outstanding balances. Zaakiy explains these verified findings but does not
invent amounts, severity, causes, or accounting entries.

## Export

The PDF endpoint uses the same backend report result as the JSON endpoint and
includes the selected period, scope, summary, trends, findings, and the
accounting disclaimer. PDF export is also protected by `accounts.view`.
