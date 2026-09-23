# Phase 1 Release Smoke Checklist

Run this checklist against the deployed environment with deterministic test
accounts and a selected branch. Record the branch, user, date, and API/build
version used.

- Login with an active user; verify invalid and inactive logins are rejected.
- Select an authorized branch; verify Owners, Properties, Agreements, Accounts,
  Reports, Dashboard, and Audit show only that branch.
- Create an Owner, Property, and Owner Agreement; submit, approve, and commence
  it; verify generated numbers, schedule, history, and audit entries.
- Create a Tenant Agreement from covered available property; verify overlap and
  out-of-coverage requests are rejected.
- Post cash, cheque, and bank-transfer payments; verify direction, metadata,
  receipt/voucher, allocation, idempotency, and outstanding balances.
- Deposit and clear a cheque; verify the original transaction and number remain
  unchanged. Void one posted payment and verify balance restoration.
- Exercise hold/resume, extension, renewal, termination, and scheduled lifecycle
  processing where the release fixture supports them.
- Create a petty-cash entry, quotation/invoice payment, and work order payment;
  verify branch scope and financial traceability.
- Reconcile Dashboard attention values with the corresponding Reports summaries.
- Open Audit Trail; verify representative customer, property, agreement,
  payment, cheque, void, and work-order events.
- Logout; verify protected API calls are no longer usable.

Known deferred items: explicit All-Branches mode, parallel database race
harness, advanced component-test depth, report exports, cheque reconciliation,
bounced-cheque accounting policy, administration mutation APIs, attachments,
notifications, procurement, AI, and General Ledger.
