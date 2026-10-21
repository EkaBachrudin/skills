# Backend Bugs Workflow

Detailed procedure for the `backend-bugs` skill. `SKILL.md` holds the binding
rules; this file holds the reproduction and diagnosis procedure.

## 1. Scope and environment

Record:

- Target: endpoint, job, worker, or flow.
- Environment: local, CI, staging, or production.
- Expected behavior: the contract, acceptance criterion, or invariant.
- Actual behavior: status code, response body, log, state, or side effect.
- Frequency: always, intermittent, load-dependent, time-dependent.
- Constraints: no writes, no production data, time budget.

If the issue is intermittent, record the observed rate and the conditions under
which it appears. "Sometimes" is not a reproduction.

## 2. Map flows and invariants

Write the rules that must hold:

- "Each payment intent is captured at most once."
- "A tenant can only see its own records."
- "A failed job does not leave a partial row."
- "The sum of ledger entries equals the account balance."
- "A retried request produces the same result as the original."

Trace the flow:

1. Entry point and validation.
2. Authorization and tenant scope.
3. Service logic and state transitions.
4. Database reads, writes, transactions, and constraints.
5. Queue or worker handoff.
6. External calls and retries.
7. Response mapping and error handling.

Mark the point where each invariant is supposed to be enforced.

## 3. Static inspection

Read the actual path, not just the happy-path handler. Look for:

- Check-then-act sequences without a transaction or lock.
- Reads followed by writes based on stale values.
- Missing unique constraints or idempotency keys.
- Transactions that span network calls.
- `try/catch` that logs and continues, or returns success after failure.
- Error responses that do not match the documented contract.
- Partial writes across multiple repositories or services.
- Retries that repeat non-idempotent side effects.
- Queue consumers that acknowledge before work is durable.
- Locks, streams, cursors, or connections not released on every path.
- Time comparisons without a time zone or with a DST assumption.
- Pagination based on a non-unique or mutable sort key.
- Integer, decimal, or currency rounding mismatches.
- Case-sensitive or Unicode-normalization assumptions.

Record exact `file:line` locations for each candidate.

## 4. Reproduce

Start with the smallest possible reproduction.

### Deterministic tests

- Unit test the pure function with the failing input.
- Integration test the handler with a real database or a faithful test double.
- Use fixed seeds for random data.
- Freeze or inject the clock for time-dependent bugs.
- Control the sort order for pagination and concurrency tests.

### Concurrency

- Run two or more requests with a barrier so they overlap.
- Repeat the test enough times to expose the race.
- Use separate connections or transactions to avoid accidental serialization.
- Check the final state, not just the response codes.
- Record the interleaving or logs that show the lost update or double write.

### Failure injection

- Make a dependency fail or time out after a partial write.
- Drop the connection after the database commit but before the response.
- Retry the same request with the same idempotency key.
- Kill a worker mid-job and observe recovery.
- Simulate a duplicate queue delivery.

### Data shape

- Empty, null, zero, negative, maximum, duplicate, and malformed inputs.
- Unicode, case variants, leading/trailing spaces, and long strings.
- Different locales, time zones, and currency scales.
- Large pages, deep offsets, and mutable sort keys.

Do not use production data. If you must use a production-shaped fixture, redact
and minimize it.

## 5. Isolate the root cause

Use a bisect-style approach:

1. Confirm the failure at the outermost layer.
2. Add logging or assertions around the suspected invariant.
3. Move inward one layer at a time.
4. Identify the first point where the observed state diverges from the expected
   state.
5. Check whether the cause is logic, ordering, data, configuration, or contract.

Separate:

- Trigger: what starts the failure.
- Mechanism: how the state becomes invalid.
- Impact: what the user or downstream system observes.
- Root cause: the smallest code or design defect that explains the mechanism.

## 6. Fix options

Prefer the smallest fix that restores the invariant:

- Add a database constraint or unique index for uniqueness.
- Move the read and write into one transaction with the right isolation.
- Use an atomic conditional update instead of check-then-act.
- Add an idempotency key with a uniqueness constraint.
- Make retries safe or remove the retry.
- Acknowledge a queue message only after durable work.
- Return the documented error instead of swallowing it.
- Normalize time, locale, or currency at the boundary.
- Use a stable, unique sort key for pagination.

For each option, record:

- What invariant it restores.
- Side effects and migration risk.
- Whether it is a local fix or a design change.
- How to roll it back.

Do not silently broaden the scope into a refactor. Note related issues separately.

## 7. Regression tests

Add a test that fails before the fix and passes after:

- Reproduce the exact trigger.
- Assert the invariant, not just the status code.
- Cover the boundary values and the concurrent interleaving.
- Use deterministic seeds and a controlled clock.
- Keep the test small enough to run in CI.

If the fix is operational rather than code, add a check or alert that detects
the invariant violation.

## 8. Report

Write incrementally to:

```text
.backend-skills/reports/<run-id>/bugs.md
```

Use `YYYYMMDD-HHMMSS` for `<run-id>`. Include:

- Scope, environment, expected and actual behavior.
- Reproduction steps or the failing test.
- Frequency and conditions.
- Findings ordered by severity.
- Root cause with `file:line`.
- Fix, regression test, and verification.
- Blast radius and any data repair needed.
- Unreproduced issues and what was tried.

## 9. Stop conditions

Stop and report when:

- The bug is reproduced and the first invariant violation is identified.
- The next step requires a code change or production access.
- The issue is intermittent and the remaining work is instrumentation.
- The fix would require a design decision or data migration.
- You find a security issue; switch to the security review process.
