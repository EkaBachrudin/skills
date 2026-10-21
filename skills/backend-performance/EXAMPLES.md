# Backend Performance Examples

Illustrative findings. Numbers are examples, not claims about any real system.

## Example 1: N+1 query in an orders endpoint

**Symptom**

`GET /api/orders` has p95 latency of 1.8 s with 50 orders per page. Database
CPU is low, but the endpoint issues 51 queries per request.

**Evidence**

```text
query_count_per_request=51
orders_rows=50
p95_latency_ms=1800
```

```ts
// src/modules/orders/order.service.ts:84
const orders = await orderRepository.find({ where: { tenantId } });
for (const order of orders) {
  order.customer = await customerRepository.findOne(order.customerId);
}
```

**Finding**

- Severity: High
- Confidence: Confirmed
- Impact: Latency grows linearly with page size; database round trips dominate.
- Root cause: Lazy per-row lookup inside a loop.
- Fix: Batch-load customers with a single query keyed by the distinct customer
  IDs, or use a join/relation load strategy supported by the ORM.
- Verification: Query count per request drops from 51 to 2; p95 improves under
  the same workload.

## Example 2: Missing composite index

**Symptom**

The order history query is slow for large tenants.

**Evidence**

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT id, total, created_at
FROM orders
WHERE tenant_id = $1
ORDER BY created_at DESC
LIMIT 50;
```

```text
Seq Scan on orders  (rows=2400000) (actual time=0.031..842.117 rows=50)
Sort  (actual time=842.118..842.119 rows=50)
```

**Finding**

- Severity: High
- Confidence: Confirmed
- Impact: p95 grows with table size and tenant data volume.
- Root cause: No index supports the filter and sort together.
- Fix: Add a composite index on `(tenant_id, created_at DESC)` after checking
  write cost and migration safety.
- Verification: Plan changes to an index scan; buffers and p95 are recorded
  before and after.

## Example 3: Unbounded fan-out exhausts the connection pool

**Symptom**

A batch endpoint works for 10 items but times out for 500.

**Evidence**

```ts
// src/modules/exports/export.service.ts:41
const results = await Promise.all(
  ids.map((id) => repository.loadAggregate(id)),
);
```

```text
pool_size=10
in_flight_queries=500
pool_wait_p95_ms=9200
```

**Finding**

- Severity: High
- Confidence: Confirmed
- Impact: Pool starvation affects unrelated requests; timeouts cascade.
- Root cause: Unbounded concurrency over a shared pool.
- Fix: Process in bounded batches or use a concurrency limiter sized to the
  pool and downstream capacity.
- Verification: In-flight queries stay at or below the limit; pool wait time
  and p95 drop under the same batch size.

## Example 4: Cache key ignores tenant

**Symptom**

After a cache warm-up, one tenant occasionally sees another tenant's data.

**Finding**

- Severity: Critical
- Confidence: Confirmed
- Impact: Cross-tenant data exposure plus incorrect cached responses.
- Root cause: Cache key uses only resource ID, not tenant ID.
- Fix: Include tenant and authorization scope in the key; invalidate affected
  entries; add a regression test.
- Note: This is also a security issue. Report it through the security process;
  do not treat it as a performance-only finding.

## Report template

```markdown
# Backend Performance Report

- Run ID:
- Date:
- Target:
- Environment:
- Workload:
- Constraints:

## Baseline

| Metric | Value | Notes |
| --- | --- | --- |
| p50 latency | | |
| p95 latency | | |
| p99 latency | | |
| Throughput | | |
| Error rate | | |
| CPU | | |
| Memory | | |

## Findings

### PERF-001: <title>

- Severity:
- Confidence:
- Location:
- Endpoint or job:
- Evidence:
- Impact:
- Root cause:
- Recommended fix:
- Risk and rollback:
- Verification:

## Prioritized actions

1.
2.
3.

## Unverified or blocked

-

## Appendix

- Commands:
- Query plans:
- Profiles:
```
