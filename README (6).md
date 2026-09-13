# Comm-Log Send Reconciliation

## Objective

Reproduce Finance's reported `target_base` for:

- **Merchant:** 501
- **Period:** October 2026
- **Scope:** Campaign communications / Diwali campaigns
- **Finance target:** **22**

The goal is not only to reach 22, but to show the investigation and explain why a straightforward query does not reconcile immediately.

## Final result

**`target_base = 22`**

## Reconciliation bridge

| Step | Description | Result |
|---:|---|---:|
| 0 | Naive `COUNT(DISTINCT customer_id)` | 25 |
| 1 | Apply campaign eligibility | 21 |
| 2 | Keep successful deliveries | 21 |
| 3 | Correct standalone-campaign event grain | 22 |
| **Final** | **Finance target_base** | **22** |


## Key findings

### 1. Raw log rows are not automatically reportable

Campaign 9004 has communication-log rows but its creation status is `approval_awaiting`. The reporting eligibility rule requires a finalized creation status and `processing_status = 'processed'`.

### 2. Retry campaigns must be treated as one underlying communication

A campaign with a `parent_id` is a retry of the parent campaign. A customer can therefore appear across multiple campaign IDs while still representing one underlying communication.

### 3. Standalone campaigns have a different grain

Campaign 9101 has no parent and no child, making it a standalone communication. Customer C20 appears twice, and both sends are legitimate events. A global `COUNT(DISTINCT customer_id)` therefore undercounts the final metric by one.


Expected output:

```text
22
```

If using the supplied CSVs, the same logic can be reproduced in pandas/SQLite after loading the two CSV files into tables.

## Note

The repository is designed to show the analytical investigation, not just the final answer. The final SQL follows the reporting definition: retry chains are deduplicated by customer within the underlying communication, while true standalone send events remain event-level.
