# Payments & Accounting

## Scope

Baithul Madeena is not assumed to be a full general-ledger accounting system unless implemented later.

This document defines operational financial controls required by the ERP.

---

## Payment Direction

Use explicit direction:

```text
inward
outward
```

Examples:

```text
Tenant payment → inward
Owner payout → outward
```

---

## Payment Modes

Supported modes:

```text
cash
cheque
bank_transfer
```

Suggested mode-specific details:

### Cash

```text
received_by / paid_by
transaction_date
```

### Cheque

```text
cheque_no
cheque_date
bank_name
status
```

Possible cheque statuses:

```text
received
deposited
cleared
bounced
cancelled
```

### Bank transfer

```text
bank_reference
bank_name
transfer_date
```

---

## Receipts

### Cash receipt

Generated for inward revenue.

Must contain enough information to trace:

```text
receipt_no
branch
payment
payer
amount
date
payment_mode
reference
created_by
```

### Purchase receipt

Generated for outward revenue/payment.

Must contain enough information to trace:

```text
receipt_no
branch
payment
payee
amount
date
payment_mode
reference
created_by
```

---

## Posting

Treat posting as a controlled state transition.

Recommended concepts:

```text
draft
posted
void
```

Once posted:

- amount changes should normally be prohibited;
- party changes should normally be prohibited;
- receipt number should not be reused;
- corrections should use void/reversal plus replacement.

---

## Idempotency

Financial endpoints should be safe against accidental double submission.

Consider idempotency keys for:

- payment posting;
- receipt issuance;
- purchase receiving;
- other financial mutations.

At minimum, enforce business-level duplicate checks.

---

## Agreement Reconciliation

Tenant and owner agreement payments should be reconcilable to installments.

Support:

- full installment payment;
- partial payment;
- payment covering multiple installments, if required;
- outstanding balance;
- overdue balance.

Do not silently distribute amounts across installments without a documented allocation rule.

---

## Numbering

Financial document numbering should be deterministic.

Example format:

```text
BR01-CR-2026-000001
BR01-PR-2026-000001
```

Actual format is a product decision.

The critical requirement is uniqueness and non-reuse.

The current operational ledger uses `account_transactions` as the authoritative
posted money record. Agreement payments derive direction from context: tenant
payments are inward and owner payments are outward. The transaction's document
number is the receipt/voucher record; separate duplicate receipt tables are not
created.

Agreement payment posting locks the agreement/installments, re-reads the
outstanding balance, allocates the complete payment atomically, and accepts an
`Idempotency-Key`. Payment allocation totals must equal the posted amount and
cannot exceed installment outstanding balances.

Payment mode metadata is structured. Cash uses a server-generated default
remark (`Cash Payment <sequence>`); cheque requires cheque number/date; bank
transfer requires bank reference/transfer date. Irrelevant mode fields are
cleared before persistence.

---

## Cancellation / Void

Store:

```text
voided_by
voided_at
void_reason
replacement_document_id nullable
```

Never delete a posted receipt to "fix" a transaction.

Voiding is an explicit `POST /api/v1/accounts/transactions/{id}/void` action
requiring a reason. It preserves the original document number and source
record, reverses installment allocations atomically, and excludes the voided
entry from petty-cash balances. Posted and voided records are immutable.
