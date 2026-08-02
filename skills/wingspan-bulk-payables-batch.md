---
name: Bulk-import payables with batches
description: Create a payable batch, add items, and process it to import many contractor
  payments in one run.
api: openapi/wingspan-payments-openapi-original.yml
operations: [createPayableBatch, createPayableBatchItem, listPayableBatchItems, getPayableBatch, updatePayableBatch]
generated: '2026-07-21'
method: generated
---

# Bulk-import payables with batches

Ground rules: bearer-token auth, staging first, `{"error": "..."}` envelope, 100 write/s
rate limit — batch APIs exist precisely so you do not hammer the single-payable endpoint
(see `rate-limits/wingspan-rate-limits.yml`).

## Steps

1. **Open a batch** — `createPayableBatch` to create the container.
2. **Add items** — `createPayableBatchItem` once per payable row (collaborator, amount, line items). Throttle below the write rate limit; queue and retry `429`s with jittered backoff.
3. **Audit the staged rows** — `listPayableBatchItems` and reconcile the count/amounts against your source file before processing.
4. **Process the batch** — `updatePayableBatch` to change the batch status to processing/executed as documented. This creates real payables in production.
5. **Monitor** — `getPayableBatch` for batch status, and per-item status via `listPayableBatchItems`; failed items report through the same `{"error"}` envelope.
