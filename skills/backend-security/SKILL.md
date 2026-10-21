---
name: backend-security
version: 1.0.0
description: >-
  Use when the user asks to audit or harden backend security: authentication,
  authorization and IDOR, injection, SSRF, secret exposure, input validation,
  rate limiting, dependency vulnerabilities, or trust-boundary issues. Produces
  a severity-ranked security report with reproduction evidence. Triggers on
  "backend security", "security audit", "check for vulnerabilities", or
  "review authz". Not for latency tuning unless the issue is security-related.
---

# Backend Security

A defensive, feature-scoped security review. Find trust-boundary mistakes,
prove them safely, and report them with enough evidence to fix them.

## Use this skill when

- Reviewing authentication, sessions, tokens, authorization, ownership, tenant isolation, or admin routes.
- Auditing input handling for injection, deserialization, or file upload risk.
- Checking secret exposure, unsafe outbound requests, SSRF, rate limits, or abuse controls.
- Assessing dependency and configuration risk in a backend service.

## Do not use this skill for

- Performance tuning with no security dimension, or correctness bugs with no security impact.
- Running an unapproved penetration test or attacking a live system.
- Producing exploit tooling or payloads beyond a minimal proof of concept.

## Authorization and safety rules

1. **Only test what you are authorized to test.** Record the target and permission in the report.
2. **Read-only by default.** Do not modify code, data, configuration, or permissions unless asked.
3. **Minimal proof.** Stop at the first safe confirmation. Do not exfiltrate data, persist access, pivot laterally, or run denial-of-service tests.
4. **Use synthetic data and redact.** Prefer test accounts and records; replace tokens, cookies, keys, emails, and identifiers with stable placeholders.
5. **Respect production.** Do not probe production without explicit written permission. Prefer local or staging.
6. **Stop on incident signals.** If you find an active compromise or real data exposure, stop testing and report immediately.

## Review workflow

1. **Scope and threat model.** Identify assets, actors, entry points, trust boundaries, and the data each flow can reach.
2. **Authentication.** Check identity proof, session or token lifecycle, revocation, and credential storage.
3. **Authorization.** Check function-level and object-level access, tenant isolation, and default-deny behavior.
4. **Input handling.** Check validation, canonicalization, query construction, templates, deserialization, and file handling.
5. **Data protection.** Check secrets, logs, error responses, backups, and data returned by APIs.
6. **Platform and abuse.** Check SSRF, allowlists, metadata endpoints, webhooks, CORS, headers, rate limits, quotas, and replay protection.
7. **Dependencies and configuration.** Check known-vulnerable packages, debug settings, default credentials, and overly broad permissions.
8. **Report.** Write findings incrementally to `.backend-skills/reports/<run-id>/security.md`.

## Severity and confidence

- `Critical`: full compromise, cross-tenant access, auth bypass, or mass data exposure.
- `High`: privilege escalation, sensitive data access, or a directly exploitable injection.
- `Medium`: limited access, conditional exploitation, or a meaningful defense gap.
- `Low`: hardening issue with limited direct impact.
- `Info`: observation or best-practice note.

Confidence: `Confirmed` (reproduced safely), `Likely` (clear code path, no runtime confirmation), or `Unverified` (plausible, needs access you do not have).

## Evidence standard

Each finding includes:

- ID, title, severity, and confidence
- Affected endpoint, job, or trust boundary; location (`file:line`) and configuration
- Minimal reproduction with synthetic data
- Impact and who can exploit it
- Root cause and recommended fix, plus a defense-in-depth option
- Verification step and regression test

## Resources

- [WORKFLOW.md](WORKFLOW.md) — detailed procedure and safe verification.
- [CHECKLIST.md](CHECKLIST.md) — coverage checklist by control area.
- [EXAMPLES.md](EXAMPLES.md) — worked findings and report template.

## Done when

- Every entry point in scope is mapped to a trust boundary.
- Authentication and object-level authorization are explicitly checked.
- Every finding has severity, confidence, evidence, and a fix.
- The report is written and secrets are redacted.
