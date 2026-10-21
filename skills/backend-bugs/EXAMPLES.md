# Backend Bugs Examples

Illustrative findings. Names, IDs, and numbers are examples, not claims about a
real system.

## Example 1: Lost update on account balance

**Symptom**

Two concurrent debits both succeed, but only one is reflected in the balance.

**Expected**

`balance = 100 - 30 - 20 = 50`.

**Actual**

`balance = 80`; one debit is missing even though both requests returned `200`.

**Evidence**

```ts
// src/modules/accounts/account.service.ts:52
const account = await accountRepository.findById(id);
if (account.balance < amount) throw new InsufficientFundsError();
account.balance -= amount;
await accountRepository.save(account);
```

```text
request A read balance=100
request B read balance=100
request A wrote balance=70
request B wrote balance=80
```

**Finding**

- Severity: Critical
- Confidence: Confirmed
- Impact: Money is created or lost; ledger and balance diverge.
- Root cause: Read-modify-write without a transaction, row lock, or atomic
  conditional update.
- Fix: Use an atomic conditional update (`balance = balance - :amount WHERE
  balance >= :amount`) or a transaction with `SELECT ... FOR UPDATE`; add a
  ledger entry in the same transaction.
- Regression test: Fire two concurrent debits and assert the final balance and
  ledger entries.
- Verification: The invariant holds across repeated concurrent runs.

## Example 2: Non-idempotent retry creates duplicate orders

**Symptom**

A client retries after a timeout and receives two orders.

**Expected**

One order per idempotency key.

**Actual**

Two orders with the same payload and different IDs.

**Evidence**

```ts
// src/modules/orders/order.service.ts:31
const order = await orderRepository.create(body);
await paymentClient.charge(order.id, body.amount);
return order;
```

**Finding**

- Severity: High
- Confidence: Confirmed
- Impact: Duplicate charges and fulfillment.
- Root cause: No idempotency key and no uniqueness constraint; the retry creates
  a new order before the payment call.
- Fix: Require an idempotency key, store it with a unique constraint scoped to
  the tenant and operation, and return the original result for a replay.
- Regression test: Send the same key twice, including a timeout after commit,
  and assert one order and one charge.
- Verification: Replays return the original order; a different key creates a new
  order.

## Example 3: Swallowed exception leaves a partial write

**Symptom**

The API returns `201`, but the audit record and notification are missing.

**Evidence**

```ts
// src/modules/payments/payment.service.ts:88
await paymentRepository.insert(payment);
try {
  await auditRepository.insert(event);
  await notificationService.send(payment.id);
} catch (error) {
  logger.warn("post-payment side effect failed", error);
}
return payment;
```

**Finding**

- Severity: High
- Confidence: Confirmed
- Impact: Missing audit trail and customer notification; the caller believes the
  whole operation succeeded.
- Root cause: Side effects are treated as best-effort and failures are swallowed.
- Fix: Write the audit record in the same transaction as the payment; move the
  notification to a durable outbox or queue with retries; return a truthful
  result if a required side effect fails.
- Regression test: Make the audit insert fail and assert the payment is rolled
  back or the operation is reported as failed.
- Verification: A forced failure leaves no partial state and surfaces the error.

## Example 4: Pagination duplicates and skips rows

**Symptom**

A user exporting all invoices sees duplicates and misses records.

**Evidence**

```sql
SELECT id, created_at
FROM invoices
ORDER BY created_at DESC
LIMIT 50 OFFSET 100;
```

**Finding**

- Severity: Medium
- Confidence: Confirmed
- Impact: Incomplete exports and duplicate processing.
- Root cause: Offset pagination over a mutable, non-unique sort key.
- Fix: Use keyset pagination with a stable, unique tiebreaker such as
  `(created_at, id)`; keep the filter consistent across pages.
- Regression test: Insert rows with equal timestamps and mutate data between
  pages; assert no duplicates and no missing rows.
- Verification: The export count matches the source set.

## Example 5: Time-zone shift on a date-only field

**Symptom**

A subscription scheduled for `2026-03-29` is charged on `2026-03-28` for some
users.

**Finding**

- Severity: Medium
- Confidence: Likely
- Impact: Off-by-one billing date around DST and time-zone boundaries.
- Root cause: A date-only value is parsed as local midnight and converted to
  UTC, then shifted again for display.
- Fix: Store date-only values as dates, not timestamps; do date arithmetic in
  the business time zone; test DST transitions explicitly.
- Regression test: Cover the DST transition dates for each supported time zone.
- Verification: The scheduled date is identical in storage, API, and display.

## Report template

````markdown
# Backend Bug Report

- Run ID:
- Date:
- Target:
- Environment:
- Expected behavior:
- Actual behavior:
- Frequency:

## Reproduction

1.
2.
3.

```text
<command, request, or test output>
```

## Findings

### BUG-001: <title>

- Severity:
- Confidence:
- Affected endpoint or flow:
- Location:
- Expected:
- Actual:
- Minimal reproduction:
- Evidence:
- Root cause:
- Impact and blast radius:
- Recommended fix:
- Regression test:
- Verification:
- Data repair, if any:

## Unreproduced or blocked

-

## Appendix

- Logs:
- State snapshots:
- Related issues:
````
