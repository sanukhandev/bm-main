# Maintenance & Inventory

## Work Orders

A work order can be raised against:

- a property; or
- a rentable component such as a unit, room, area, or bed.

Suggested fields:

```text
work_order_no
branch_id
property_id
rentable_component_id
title
description
priority
status
service_charge
assigned_to
opened_at
completed_at
```

---

## Work Order Cost

Total work-order cost can include:

```text
service charge
+ inventory item cost
+ optional external/vendor cost
+ optional other approved charges
```

Store cost components separately. Do not only store a final total.

---

## Inventory Items

Suggested fields:

```text
sku
name
description
unit_of_measure
reorder_level
active
```

Examples of units:

```text
piece
box
meter
liter
kg
```

---

## Stock Movements

Stock quantity must be derived from or reconciled against auditable movement records.

Movement fields may include:

```text
branch_id
inventory_item_id
movement_type
quantity
unit_cost
reference_type
reference_id
occurred_at
created_by
notes
```

Positive/negative sign conventions must be documented.

---

## Purchase Order Flow

Suggested flow:

```text
draft
  ↓
approved
  ↓
ordered
  ↓
partially_received
  ↓
received
```

Receiving should:

1. validate remaining receivable quantity;
2. create receipt/receiving record;
3. create stock-in movement;
4. update PO received quantity;
5. update PO status;
6. run in one database transaction.

---

## Work Order Consumption

When an inventory item is used:

1. validate work order;
2. validate branch;
3. validate stock availability if negative stock is disallowed;
4. create work-order item usage;
5. create stock-out movement;
6. calculate captured cost;
7. commit atomically.

---

## Return to Inventory

Unused items may be returned.

A return should:

- reference the original work-order consumption where possible;
- not return more than the quantity consumed;
- create a stock-in movement;
- retain audit trail.

---

## Scrap / Write-Off

Inventory can be cleaned up when items are scrapped.

Scrap flow should require:

```text
item
quantity
reason
date
authorized user
optional attachment/photo
```

Scrap creates a stock-out/write-off movement.

For material adjustments, consider requiring approval.

---

## Stock Valuation

If inventory valuation is required, choose and document one method, for example:

```text
weighted average
FIFO
standard cost
```

Do not mix valuation methods unintentionally.

If valuation is out of scope initially, still store enough data to support future reporting where practical.

---

## Negative Stock

Recommended default: disallow negative stock.

If the business permits negative stock, make it an explicit configuration and expose exceptions in reports.

---

## Inventory Branch Scope

Inventory should be branch-scoped unless the business introduces central warehouse transfers.

If inter-branch transfer is later added, it must create two-sided transfer records, not silently change branch ownership.
