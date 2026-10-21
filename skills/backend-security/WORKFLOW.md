# Backend Security Workflow

Detailed procedure for the `backend-security` skill. `SKILL.md` holds the
binding safety rules; this file holds the review procedure.

## 1. Scope and authorization

Record:

- Target: service, endpoints, jobs, or feature.
- Environment: local, CI, staging, or production.
- Authorization: who approved testing and what is in scope.
- Accounts: test users, roles, tenants, and permissions available.
- Constraints: no writes, no load, no production probing, time budget.

If authorization is unclear, perform a code-only review and mark runtime
findings as `unverified`. Do not send probing traffic to a system you do not
own or have permission to test.

## 2. Threat model

Write a short model before reading code:

- Assets: data, credentials, money, admin actions, availability.
- Actors: anonymous user, authenticated user, tenant member, tenant admin,
  service account, internal operator, attacker with a stolen token.
- Entry points: HTTP routes, GraphQL operations, webhooks, queues, scheduled
  jobs, CLI commands, file imports.
- Trust boundaries: public to private network, tenant to tenant, user to admin,
  service to service, application to database, application to third parties.
- Invariants: "a user can only read their own tenant's records", "only an admin
  can change roles", "a payment can only be captured once".

Use the model to prioritize. A route that reaches sensitive data across a trust
boundary deserves more attention than a static health endpoint.

## 3. Map entry points and trust boundaries

Build a table:

| Entry point | Auth | Roles | Data reached | Boundary | Notes |
| --- | --- | --- | --- | --- | --- |
| `GET /api/invoices/:id` | Bearer | member | invoice, customer | tenant | object ID in path |
| `POST /api/admin/users` | Bearer | admin | user, roles | user to admin | role assignment |
| `POST /webhooks/payment` | signature | service | payment | third party | replay risk |

Trace each entry point to its handler, service, data access, and outbound calls.
Record the exact files and lines.

## 4. Authentication

Check:

- Password storage uses a modern adaptive hash with per-user salt.
- Session cookies are `HttpOnly`, `Secure`, `SameSite` where appropriate.
- Tokens have bounded lifetime, audience, issuer, and revocation behavior.
- Refresh tokens rotate or are protected against replay.
- Login, reset, and invitation flows do not leak whether an account exists.
- MFA or step-up authentication protects sensitive actions where required.
- Service-to-service identity is verified, not assumed from network location.
- Logout and role changes invalidate active sessions where the design requires.

Safe verification: use test accounts. Do not brute force, credential stuff, or
test against real user accounts.

## 5. Authorization

Check both function-level and object-level access.

### Function-level

- Every sensitive route has an explicit permission check.
- Admin routes are not protected only by a hidden UI or a client-side flag.
- Default behavior is deny, not allow.
- Role checks cannot be bypassed by omitting a parameter or changing a method.
- Bulk and export endpoints enforce the same rules as single-item endpoints.

### Object-level

- The handler verifies that the authenticated subject owns or may access the
  object, not just that the object exists.
- Tenant ID is derived from the authenticated context, not from a client-supplied
  body or query parameter.
- Nested resources verify the parent-child relationship.
- Deleted, archived, or transferred objects do not remain accessible.
- List endpoints filter by authorization scope, not just by a requested filter.

Safe verification: create two test accounts in separate tenants. Request the
same object ID as the other account and expect a denial. Stop at the first
confirmation; do not enumerate IDs or download unrelated data.

## 6. Input handling and injection

Check:

- Inputs are validated against an allowlist of shape, type, length, and range.
- Canonicalization happens before validation and authorization.
- Queries use parameter binding or safe ORM APIs; raw string concatenation is
  flagged.
- Shell commands avoid untrusted input; use argument arrays, not shell strings.
- Templates auto-escape by default; raw rendering is explicit and justified.
- Deserialization does not accept arbitrary classes or untrusted object graphs.
- XML, YAML, CSV, and archive parsers have limits and safe defaults.
- File uploads validate content, size, type, and storage location.
- Redirects and proxy targets use allowlists, not free-form URLs.
- Errors do not echo secrets, stack traces, or raw query text to users.

Report the pattern and the fix. Do not include weaponized payloads in the report.

## 7. Data protection and secrets

Check:

- Secrets are loaded from a secret manager or environment, not committed files.
- Secrets are not present in logs, traces, error responses, or build artifacts.
- API responses return only the fields the caller needs.
- Sensitive fields are masked or omitted in list and export endpoints.
- Backups, temp files, and debug endpoints have appropriate access control.
- Personal data has retention and deletion behavior that matches policy.
- Encryption in transit and at rest is configured where required.

If a secret is found, do not copy it into the report. Record its location, type,
and a redacted fingerprint, then recommend rotation and removal from history.

## 8. Outbound requests, SSRF, and file handling

Check:

- User-controlled URLs are validated against an allowlist of schemes, hosts,
  ports, and resolved IP ranges.
- Redirects are not followed blindly to internal addresses.
- Cloud metadata endpoints and link-local addresses are blocked.
- Webhook destinations are verified and signed where applicable.
- File fetches have size, content-type, and timeout limits.
- Uploaded files are stored outside the web root and served with safe headers.
- Archive extraction prevents path traversal and decompression bombs.
- Deserialization and template rendering do not execute untrusted content.

Safe verification: use a test URL you control or a local test server. Do not
probe internal production services or metadata endpoints.

## 9. Abuse and rate limiting

Check:

- Login, reset, invitation, and OTP flows have per-account and per-IP limits.
- Expensive endpoints have quotas, pagination limits, and timeouts.
- Idempotency keys protect payment, order, and retry-prone operations.
- Replay protection exists for signed webhooks and one-time tokens.
- Enumeration surfaces do not reveal valid accounts, IDs, or resources.
- Fraud-sensitive actions have velocity checks and audit trails.
- Rate-limit responses do not leak internal details.

## 10. Dependencies and configuration

Check:

- Dependency versions are pinned and scanned by the project's existing tool.
- Known-vulnerable packages have an owner and a remediation plan.
- Debug mode, verbose errors, and development endpoints are off in production.
- Default credentials and sample accounts are removed.
- CORS origins are explicit; wildcard plus credentials is flagged.
- Security headers are present at the edge or application layer.
- Database and object-storage permissions follow least privilege.
- CI secrets are scoped and not exposed to pull requests from forks.

Do not run a full vulnerability scanner against production. Use the project's
approved tooling and environment.

## 11. Report

Write incrementally to:

```text
.backend-skills/reports/<run-id>/security.md
```

Use `YYYYMMDD-HHMMSS` for `<run-id>`. Include:

- Scope, authorization, environment, and limitations.
- Threat model and entry-point table.
- Findings ordered by severity.
- Minimal reproduction with synthetic data and redacted evidence.
- Root cause, fix, defense-in-depth option, and verification.
- Unverified areas and what access would be needed.
- A short "do not ship" list for Critical and High findings.

Redact tokens, cookies, keys, emails, personal data, and production identifiers.

## 12. Stop conditions

Stop and report when:

- You have confirmed a Critical or High finding; do not continue exploiting it.
- The next step would require writing data or probing production.
- You encounter real user data or signs of an active incident.
- The remaining scope needs authorization, credentials, or a different environment.
- Additional testing would not change the top remediation actions.
