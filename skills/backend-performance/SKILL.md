---
name: backend-performance
version: 1.0.0
description: >-
  Use when the user asks to profile, diagnose, or improve backend performance:
  slow endpoints, high latency, N+1 queries, missing indexes, memory or CPU
  growth, caching, concurrency, payload size, or load-test results. Produces an
  evidence-backed report with measurements and prioritized fixes. Triggers on
  "backend performance", "why is this endpoint slow", "optimize this query",
  or "profile the API". Not for security audits or correctness bugs.
---

# Backend Performance

Measure first, optimize second. A performance claim without a baseline,
a measurement, and a verification step is a guess.

## Use this skill when

- An endpoint is slow or its p95/p99 latency regressed.
- A query is suspected of N+1, a missing index, or a full scan.
- Memory or CPU grows under load, or a worker falls behind.
- A cache, pool, queue, or concurrency change needs validation.
- A load test needs interpretation and a prioritized action list.

## Do not use this skill for

- Security vulnerabilities or abuse cases. Route those to a security review.
- Correctness bugs that have no performance symptom. Route those to a bug review.
- Micro-optimizing code without a user-visible or capacity symptom.

## Operating rules

1. **Read-only by default.** Inspect and measure; do not edit code unless asked.
2. **Baseline before change.** Record environment, workload, p50/p95/p99, throughput, error rate, and resource use.
3. **One variable at a time.** State a hypothesis, change one thing, measure again.
4. **Respect production.** Never run load tests or stress tools against production without explicit written permission. Prefer local or staging.
5. **Evidence over intuition.** Every finding cites `file:line`, a query plan, a profile, or a command output.
6. **Separate facts from hypotheses.** Mark unmeasured causes as `unverified`.
7. **Stay bounded.** Stop when the top bottlenecks and next actions are clear; do not boil the ocean.

## Workflow

1. **Scope.** Identify endpoints, jobs, or flows; the environment; the workload; and the success metric.
2. **Baseline.** Capture latency percentiles, throughput, error rate, CPU, memory, and dependency timings.
3. **Locate.** Walk the request path: transport, application, database, cache, queue, external calls.
4. **Hypothesize.** Name one likely bottleneck and the evidence for it.
5. **Verify.** Measure before and after a targeted change; keep a control run.
6. **Prioritize.** Rank fixes by impact, effort, risk, and confidence.
7. **Report.** Write the report incrementally to `.backend-skills/reports/<run-id>/performance.md`.

## Evidence standard

Each finding includes:

- ID, title, severity, and confidence (`confirmed` / `likely` / `unverified`)
- Location (`file:line`) and affected endpoint or job
- Measurement or command output
- Impact in user-visible or capacity terms
- Root cause
- Recommended fix with risk and rollback notes
- Verification step and expected result

## Resources

- [WORKFLOW.md](WORKFLOW.md) — detailed procedure, commands, and safety gates.
- [CHECKLIST.md](CHECKLIST.md) — coverage checklist by layer.
- [EXAMPLES.md](EXAMPLES.md) — worked findings and report template.

## Done when

- A baseline and at least one measured bottleneck exist.
- Every finding has evidence and a verification step.
- The report is written and the top three actions are ranked.
