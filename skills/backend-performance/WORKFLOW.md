# Backend Performance Workflow

Detailed procedure for the `backend-performance` skill. `SKILL.md` holds the
binding rules; this file holds the operating procedure.

## 1. Scope and authorization

Record:

- Target: endpoints, jobs, workers, or flows.
- Environment: local, CI, staging, or production.
- Workload: request rate, concurrency, payload sizes, data volume.
- Success metric: p95 latency, throughput, error rate, cost, or memory.
- Constraints: no code changes, no production load, time budget.

Ask for explicit permission before:

- Running load tests against shared or production environments.
- Changing configuration, feature flags, caches, or indexes.
- Capturing profiles that may contain customer data.

If the environment is unknown, treat it as production and stay read-only.

## 2. Baseline

Capture the current state before proposing anything.

### HTTP timing

```bash
curl -sS -o /dev/null \
  -w 'dns=%{time_namelookup} connect=%{time_connect} tls=%{time_appconnect} ttfb=%{time_starttransfer} total=%{time_total}\n' \
  'https://example.test/api/orders?limit=50'
```

Run enough requests to see variance. Report p50, p95, and p99 rather than a
single best-case number.

### Database

Use the query plan for the exact query the application issues:

```sql
-- PostgreSQL
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;

-- MySQL
EXPLAIN ANALYZE SELECT ...;
```

Check for sequential scans on large tables, missing or unused indexes, sort
spills, nested loops over large row counts, and repeated queries caused by an
ORM or serializer.

### Runtime and host

Use what the environment already provides:

- Node.js: `node --cpu-prof`, `node --heap-prof`
- Python: `cProfile`, `py-spy` when available
- Go: `pprof`
- JVM: Java Flight Recorder
- Host: `top`, `vmstat 1`, `iostat -xz 1`, `ss -s`
- Load: `k6`, `hey`, or `wrk` when installed and authorized

Do not install tools into a production host. Use a local or staging copy.

## 3. Map the request path

List the stages a request or job passes through:

1. Client and network: DNS, TLS, proxy, gateway, payload size.
2. Application: routing, middleware, validation, serialization.
3. Data access: number of queries, query shape, indexes, connection pool.
4. Cache: hit rate, key design, invalidation, stampede protection.
5. Queue or worker: backlog, retries, concurrency, batch size.
6. External calls: timeouts, retries, circuit breakers, fan-out.
7. Response: payload size, compression, streaming, pagination.

State which stage the evidence points to. Avoid attributing everything to "the
database" or "the framework" without a measurement.

## 4. Static inspection

Read the code path the request actually takes. Look for:

- Queries inside loops or serializers (N+1).
- `SELECT *`, missing projections, or oversized payloads.
- Missing pagination or unbounded result sets.
- Synchronous work in a request path that could be deferred.
- Unbounded `Promise.all`, thread pools, or worker fan-out.
- Missing timeouts and retries without backoff or jitter.
- Cache keys that ignore tenant, user, locale, or version.
- Locks held across network or database calls.
- Repeated serialization, deep cloning, or large object graphs.
- Per-request construction of expensive clients, regexes, or parsers.

Record exact `file:line` locations for each candidate.

## 5. Dynamic measurement

Choose the smallest measurement that can confirm or reject the hypothesis:

- Add timing around the suspected stage in a local copy.
- Enable query logging for one representative request.
- Compare query count per request before and after a change.
- Capture a CPU profile during a load test.
- Measure cache hit rate and origin request count.
- Measure pool wait time and queue depth.

Keep the workload constant. If you change two things, you cannot attribute the
result.

## 6. Hypothesis and experiments

Use this loop:

1. Write the hypothesis and the expected measurement.
2. Record the baseline.
3. Change one variable.
4. Measure again under the same workload.
5. Keep or revert based on evidence.
6. Record the result, including negative results.

Example hypotheses:

- "The orders endpoint issues one query per order; reducing it to one query
  should cut database time by an order of magnitude."
- "The composite index on `(tenant_id, created_at)` should remove the sort and
  reduce p95 latency."
- "Limiting worker concurrency to the pool size should stop pool wait time from
  growing under load."

## 7. Prioritize fixes

Rank each recommendation by:

- Impact: expected effect on the success metric.
- Effort: code, migration, or operational work.
- Risk: regression, data migration, or rollback difficulty.
- Confidence: strength of the evidence.

Prefer fixes that are reversible and measurable. Separate quick wins from
structural work. Do not recommend a rewrite when an index, query change, or
bounded concurrency fix addresses the measured bottleneck.

## 8. Write the report

Write incrementally to:

```text
.backend-skills/reports/<run-id>/performance.md
```

Use `YYYYMMDD-HHMMSS` for `<run-id>`. Include:

- Scope, environment, workload, and constraints.
- Baseline table with p50/p95/p99, throughput, error rate, CPU, memory.
- Findings ordered by severity and impact.
- Evidence excerpts with commands or query plans.
- Prioritized actions with expected impact and verification.
- Open questions and measurements that could not be taken.

Redact secrets, tokens, personal data, and production identifiers.

## 9. Stop conditions

Stop and report when:

- The top bottlenecks are measured and ranked.
- The next experiment requires authorization or a code change.
- Additional measurement would not change the top three actions.
- The environment is unsafe for further testing.
