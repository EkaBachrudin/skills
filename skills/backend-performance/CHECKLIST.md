# Backend Performance Checklist

Use this checklist to avoid blind spots. Check only what applies, and record
evidence for every finding.

## Scope and baseline

- [ ] Target endpoints, jobs, or flows are named.
- [ ] Environment and authorization are recorded.
- [ ] Workload is defined: rate, concurrency, payload sizes, data volume.
- [ ] Success metric is defined: p95/p99, throughput, error rate, CPU, memory.
- [ ] Baseline is captured before any change.
- [ ] Baseline includes variance, not just one sample.

## Transport and gateway

- [ ] DNS, TLS, and connection setup are measured separately.
- [ ] Proxy and gateway timeouts are known.
- [ ] Compression is enabled where appropriate.
- [ ] Response payload size is measured.
- [ ] Keep-alive and connection reuse are considered.

## Application path

- [ ] Middleware does not do unnecessary blocking work.
- [ ] Validation is bounded and does not materialize huge inputs.
- [ ] Serialization does not walk large object graphs unnecessarily.
- [ ] Expensive clients, regexes, and parsers are not built per request.
- [ ] CPU-heavy work is moved off the request path when possible.
- [ ] Logging does not synchronously write large payloads per request.

## Database

- [ ] Query count per request is known.
- [ ] N+1 patterns are checked in loops, serializers, and lazy relations.
- [ ] Query plans are checked for sequential scans on large tables.
- [ ] Indexes match filter, join, and sort columns.
- [ ] Unused or duplicate indexes are identified.
- [ ] Pagination is bounded; no unbounded `findAll` or full export.
- [ ] Connection pool size and wait time are measured.
- [ ] Transactions are as short as possible.
- [ ] Lock contention is checked under concurrency.
- [ ] Batch inserts and updates are used where appropriate.

## Cache

- [ ] Cache hit rate is measured.
- [ ] Keys include tenant, user, locale, and version where relevant.
- [ ] Invalidation is defined and tested.
- [ ] Stampede protection exists for hot keys.
- [ ] Cache misses do not trigger unbounded recomputation.
- [ ] Serialization cost is acceptable for cached values.

## Concurrency and queues

- [ ] Worker concurrency is bounded and matched to downstream capacity.
- [ ] Queue depth and age are monitored.
- [ ] Retries use backoff and jitter.
- [ ] Poison messages do not loop forever.
- [ ] Batch sizes are measured, not guessed.
- [ ] Graceful shutdown drains in-flight work.
- [ ] Locks are not held across network calls.

## External calls

- [ ] Every outbound call has a timeout.
- [ ] Retries are bounded and idempotent.
- [ ] Circuit breakers or fallbacks exist for critical dependencies.
- [ ] Fan-out is bounded.
- [ ] Slow dependencies are visible in metrics or traces.
- [ ] DNS and connection reuse are considered.

## Memory and CPU

- [ ] Heap or RSS growth is measured over time.
- [ ] Large collections are streamed or paginated.
- [ ] Buffers and temporary objects are released.
- [ ] CPU profiles show the hot functions, not guesses.
- [ ] Garbage-collection pauses are checked when relevant.
- [ ] Caches and maps have eviction limits.

## Observability

- [ ] Latency is recorded as percentiles, not averages.
- [ ] Traces or logs identify the slow stage.
- [ ] Dependency timings are visible.
- [ ] Error rate is tracked alongside latency.
- [ ] Saturation signals are tracked: CPU, memory, pool, queue.

## Verification

- [ ] Each recommendation has a before/after measurement.
- [ ] The workload stayed constant during comparison.
- [ ] Negative results are recorded.
- [ ] Rollback and risk are documented.
- [ ] The report is written under `.backend-skills/reports/<run-id>/`.
