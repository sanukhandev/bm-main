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
