# Backend Security Checklist

Use this checklist to avoid blind spots. Check only what applies, and record
evidence for every finding.

## Scope and authorization

- [ ] Target and environment are named.
- [ ] Written authorization is recorded.
- [ ] Test accounts, roles, and tenants are available.
- [ ] Constraints are recorded: no writes, no load, no production probing.
- [ ] Out-of-scope systems and data are listed.
- [ ] A stop condition is agreed for active incidents or real data exposure.

## Threat model and entry points

- [ ] Assets are listed: data, credentials, money, admin actions, availability.
- [ ] Actors are listed: anonymous, user, tenant admin, service, operator.
- [ ] Entry points are mapped: HTTP, GraphQL, webhooks, queues, jobs, CLI.
- [ ] Trust boundaries are marked.
- [ ] Security invariants are written down.
- [ ] Each entry point is traced to its handler and data access.

## Authentication

- [ ] Password hashing uses a modern adaptive algorithm with per-user salt.
- [ ] Session cookies use `HttpOnly`, `Secure`, and `SameSite` where appropriate.
- [ ] Tokens have bounded lifetime, audience, issuer, and revocation.
- [ ] Refresh tokens rotate or are replay-protected.
- [ ] Login and reset flows do not reveal whether an account exists.
- [ ] MFA or step-up authentication protects sensitive actions where required.
- [ ] Service-to-service identity is verified, not trusted by network location.
- [ ] Logout and role changes invalidate sessions where required.

## Authorization

- [ ] Every sensitive route has an explicit permission check.
- [ ] Admin routes are not protected only by UI or client-side flags.
- [ ] Default behavior is deny.
- [ ] Role checks cannot be bypassed by method, parameter, or content-type changes.
- [ ] Object-level ownership is checked, not just object existence.
- [ ] Tenant ID comes from the authenticated context, not client input.
- [ ] Nested resources verify parent-child ownership.
- [ ] Deleted, archived, or transferred objects are not accessible.
- [ ] List and export endpoints filter by authorization scope.
- [ ] Bulk endpoints enforce the same rules as single-item endpoints.
- [ ] Object IDs are not enumerable in a way that leaks data.

## Input handling and injection

- [ ] Inputs are validated by allowlist: shape, type, length, range.
- [ ] Canonicalization happens before validation and authorization.
- [ ] Queries use parameter binding or safe ORM APIs.
- [ ] Raw SQL concatenation and unsafe query builders are flagged.
- [ ] Shell commands use argument arrays, not interpolated strings.
- [ ] Templates auto-escape by default; raw rendering is justified.
- [ ] Deserialization does not accept arbitrary classes or object graphs.
- [ ] XML, YAML, CSV, and archive parsers have limits and safe defaults.
- [ ] File uploads validate content, size, type, and storage location.
- [ ] Redirects and proxy targets use allowlists.
- [ ] Errors do not echo secrets, stack traces, or raw query text.

## Data protection and secrets

- [ ] Secrets come from a secret manager or environment, not committed files.
- [ ] Secrets are absent from logs, traces, errors, and build artifacts.
- [ ] API responses return only the fields the caller needs.
- [ ] Sensitive fields are masked or omitted in list and export endpoints.
- [ ] Backups, temp files, and debug endpoints are access-controlled.
- [ ] Personal data has retention and deletion behavior that matches policy.
- [ ] Encryption in transit and at rest is configured where required.
- [ ] Found secrets are redacted, rotated, and removed from history.

## Outbound requests and file handling

- [ ] User-controlled URLs are validated against scheme, host, port, and IP range.
- [ ] Redirects are not followed blindly to internal addresses.
- [ ] Cloud metadata and link-local addresses are blocked.
- [ ] Webhooks are verified and signed where applicable.
- [ ] File fetches have size, content-type, and timeout limits.
- [ ] Uploaded files are stored outside the web root.
- [ ] Files are served with safe content headers.
- [ ] Archive extraction prevents traversal and decompression bombs.
- [ ] Templates and deserialization do not execute untrusted content.

## Abuse and rate limiting

- [ ] Login, reset, invitation, and OTP flows have per-account and per-IP limits.
- [ ] Expensive endpoints have quotas, pagination limits, and timeouts.
- [ ] Idempotency keys protect payment, order, and retry-prone operations.
- [ ] Signed webhooks and one-time tokens have replay protection.
- [ ] Enumeration surfaces do not reveal valid accounts, IDs, or resources.
- [ ] Fraud-sensitive actions have velocity checks and audit trails.
- [ ] Rate-limit responses do not leak internal details.

## Dependencies and configuration

- [ ] Dependencies are pinned and scanned by approved tooling.
- [ ] Known-vulnerable packages have an owner and remediation plan.
- [ ] Debug mode and verbose errors are off in production.
- [ ] Default credentials and sample accounts are removed.
- [ ] CORS origins are explicit; wildcard plus credentials is flagged.
- [ ] Security headers are present at the edge or application layer.
- [ ] Database and object-storage permissions follow least privilege.
- [ ] CI secrets are scoped and not exposed to fork pull requests.

## Logging and monitoring

- [ ] Authentication and authorization failures are logged with context.
- [ ] Logs do not contain tokens, passwords, or personal data.
- [ ] Sensitive actions have an audit trail.
- [ ] Alerts exist for repeated denials, credential stuffing, and abuse spikes.
- [ ] Error responses have a correlation ID without internal details.

## Verification and report

- [ ] Every finding has severity and confidence.
- [ ] Every finding has a minimal, synthetic-data reproduction.
- [ ] Evidence is redacted.
- [ ] Each fix has a verification step and regression test.
- [ ] Unverified areas and required access are listed.
- [ ] Critical and High findings are called out as "do not ship".
- [ ] The report is written under `.backend-skills/reports/<run-id>/`.
