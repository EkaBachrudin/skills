# Backend Bugs Checklist

Use this checklist to avoid blind spots. Check only what applies, and record
evidence for every finding.

## Scope and reproduction

- [ ] Target endpoint, job, or flow is named.
- [ ] Environment is recorded.
- [ ] Expected behavior and actual behavior are written down.
- [ ] Frequency and conditions are recorded.
- [ ] A minimal reproduction exists, or the issue is marked unreproduced.
- [ ] The reproduction uses safe, synthetic data.
- [ ] Logs, requests, responses, and state snapshots are preserved.

## Input and edge cases

- [ ] Empty, null, missing, and zero values are tested.
- [ ] Negative, maximum, and overflow values are tested.
- [ ] Duplicate and repeated inputs are tested.
- [ ] Malformed and unexpected content types are tested.
- [ ] Unicode, case variants, and whitespace are tested.
- [ ] Long strings and large collections are tested.
- [ ] Pagination boundaries and empty pages are tested.
- [ ] Filter and sort combinations are tested.
- [ ] Concurrent duplicate submissions are tested.

## Error handling

- [ ] Every failure path returns the documented status and shape.
- [ ] Errors are not swallowed or logged without propagation.
- [ ] Partial success is not reported as full success.
- [ ] Retryable and non-retryable errors are distinguished.
- [ ] Error messages do not leak secrets or internal state.
- [ ] Timeouts and cancellations stop downstream work.
- [ ] Cleanup runs on both success and failure paths.

## Transactions and consistency

- [ ] Multi-step writes are atomic or compensated.
- [ ] Rollback works when a later step fails.
- [ ] Transactions do not span network calls unnecessarily.
- [ ] Isolation level matches the invariant.
- [ ] Counters, balances, and derived state are updated atomically.
- [ ] Database constraints enforce uniqueness and referential integrity.
- [ ] Cross-service writes have an outbox or compensation plan.
- [ ] Cache and database updates have a defined ordering.
- [ ] Reads after writes see the expected state.

## Concurrency and races

- [ ] Check-then-act sequences are atomic or locked.
- [ ] Lost-update scenarios are tested with overlapping requests.
- [ ] Double processing is prevented by a unique key or state guard.
- [ ] Deadlock retries are bounded and safe.
- [ ] Locks are released on every path.
- [ ] Lock ordering is consistent.
- [ ] Queue consumers handle duplicate delivery.
- [ ] Worker crashes do not lose or duplicate work.
- [ ] Shared mutable state is isolated per request or protected.

## Idempotency and retries

- [ ] Retried requests produce the same result.
- [ ] Idempotency keys are required for side-effecting operations.
- [ ] Idempotency records are scoped to the right tenant and operation.
- [ ] Replays do not repeat charges, emails, or job creation.
- [ ] Retry backoff is bounded and jittered.
- [ ] Poison messages do not loop forever.
- [ ] Webhook replays are rejected or deduplicated.

## Resource lifecycle

- [ ] Connections, cursors, streams, and files are closed on every path.
- [ ] Locks and leases have a timeout and owner.
- [ ] Orphaned jobs and temp files are cleaned up.
- [ ] Memory growth is bounded for large inputs.
- [ ] Background tasks are awaited or tracked.
- [ ] Graceful shutdown drains or requeues in-flight work.

## State machines

- [ ] Every transition has a guard.
- [ ] Illegal transitions are rejected.
- [ ] Terminal states cannot be re-entered.
- [ ] State and side effects are updated atomically.
- [ ] Stale or out-of-order events are rejected.
- [ ] State can be reconstructed after a crash.

## API contract

- [ ] Request schema matches the documented contract.
- [ ] Response schema matches the documented contract.
- [ ] Status codes are correct for success and failure.
- [ ] Pagination is stable and does not duplicate or skip rows.
- [ ] Sorting and filtering are deterministic.
- [ ] Versioning and deprecation behavior are respected.
- [ ] Nullability and optional fields are handled consistently.

## Time, locale, and encoding

- [ ] Timestamps store an explicit time zone or UTC.
- [ ] DST boundaries are tested.
- [ ] Date-only values are not shifted by time-zone conversion.
- [ ] Currency uses integer minor units or decimal, not binary float.
- [ ] Rounding rules are explicit and tested.
- [ ] Unicode normalization and case folding are consistent.
- [ ] Collation and sort order are intentional.
- [ ] Serialization preserves the expected types.

## Data integrity

- [ ] Orphaned rows are prevented by constraints.
- [ ] Counters and aggregates can be rebuilt.
- [ ] Soft deletes do not leak into normal queries.
- [ ] Duplicate records are prevented or reconciled.
- [ ] Backfills are idempotent and resumable.
- [ ] Data repair has a dry run and a rollback plan.

## Observability

- [ ] The failure is visible in logs or metrics.
- [ ] Correlation IDs connect the request to downstream work.
- [ ] The log includes the invariant that failed.
- [ ] Alerts exist for repeated invariant violations.
- [ ] The report includes enough evidence to reproduce.

## Regression and report

- [ ] A regression test fails before the fix and passes after.
- [ ] The test asserts the invariant, not only the status code.
- [ ] The fix is the smallest change that restores the invariant.
- [ ] Blast radius and data repair are documented.
- [ ] Unreproduced issues and attempted conditions are listed.
- [ ] The report is written under `.backend-skills/reports/<run-id>/`.
