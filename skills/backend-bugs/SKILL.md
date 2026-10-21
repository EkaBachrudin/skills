---
name: backend-bugs
version: 1.0.0
description: >-
  Use when the user asks to find or fix backend correctness bugs: race
  conditions, broken error handling, edge cases, transaction boundaries,
  idempotency, resource leaks, and API contract mismatches. Produces a
  reproducible bug report with severity and file:line evidence. Triggers on
  "backend bugs", "find bugs", "why does this API fail", or "debug this flow".
  Not for performance or security reviews.
---

# Backend Bugs

Find correctness bugs by reproducing them, not by guessing from code shape.
A bug report is only useful when someone else can reproduce it.

## Use this skill when

- An endpoint returns the wrong result or fails intermittently.
- A retry, timeout, or partial failure leaves inconsistent state.
- A race condition or concurrency bug is suspected.
- Error handling swallows failures or returns misleading responses.
- Transactions, idempotency, or state transitions look unsafe.
- An API contract or validation rule is inconsistent with the implementation.

## Do not use this skill for

- Performance tuning with no correctness symptom.
- Security vulnerabilities; route those to a security review.
- Style, naming, or refactoring work with no behavioral defect.

## Operating rules

1. **Reproduce before diagnosing.** If you cannot reproduce it, say so and list the conditions you tried.
2. **Read-only by default.** Do not edit code, data, or configuration unless the user asks for a fix.
3. **Use a safe environment.** Prefer local, CI, or staging. Do not corrupt production data to prove a bug.
4. **Minimal reproduction.** Reduce the input, sequence, and concurrency needed to trigger the failure.
5. **One variable at a time.** Change one condition, then re-run the same repro.
6. **Separate symptom from cause.** Record what the user sees, then the code path that produces it. Mark unconfirmed causes as `unverified`.
7. **Preserve evidence.** Keep the failing command, log excerpt, request, response, and relevant `file:line` references.

## Workflow

1. **Scope.** Identify the endpoint, job, or flow; the environment; and the expected behavior.
2. **Map invariants.** Write the rules that must always hold, such as "an order is charged once" or "a balance never goes negative".
3. **Inspect.** Read the handler, service, repository, transaction, and error paths. List candidate failure modes.
4. **Reproduce.** Build the smallest repro with a deterministic input, seed, or clock where possible.
5. **Isolate.** Use logs, breakpoints, or targeted tests to find the first point where the invariant breaks.
6. **Fix or recommend.** Propose the smallest safe fix; include a regression test that fails before and passes after.
7. **Report.** Write the report incrementally to `.backend-skills/reports/<run-id>/bugs.md`.

## Bug classes to consider

- Input and edge cases: empty, null, zero, negative, huge, duplicate, malformed.
- Error handling: swallowed errors, wrong status codes, partial success.
- Transactions: partial writes, missing rollback, isolation assumptions.
- Concurrency and retries: lost updates, check-then-act, double processing, unsafe replay.
- Resource lifecycle: leaks, unreleased locks, unclosed streams, orphaned jobs.
- State machines: illegal transitions, missing guards, stale state.
- API contract: schema mismatch, pagination, sorting, filtering, versioning.
- Time, locale, encoding, and data integrity: time zones, DST, rounding, Unicode, orphaned rows, counters.

## Evidence standard

Each finding includes:

- ID, title, severity, and confidence (`confirmed` / `likely` / `unverified`)
- Affected endpoint, job, or flow; expected and actual behavior
- Minimal reproduction steps or failing test, plus logs or state snapshots
- Location (`file:line`) and root cause
- Impact and blast radius
- Recommended fix, regression test, and verification

## Resources

- [WORKFLOW.md](WORKFLOW.md) — detailed procedure and reproduction techniques.
- [CHECKLIST.md](CHECKLIST.md) — coverage checklist by failure class.
- [EXAMPLES.md](EXAMPLES.md) — worked findings and report template.

## Done when

- The bug is reproduced or explicitly marked unreproduced.
- The first point of invariant violation is identified.
- A regression test and fix are proposed or implemented.
- The report is written with evidence and blast radius.
