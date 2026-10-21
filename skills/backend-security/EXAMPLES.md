# Backend Security Examples

Illustrative findings with synthetic data. Do not reuse identifiers, tokens, or
payloads against a real system.

## Example 1: IDOR on an invoice endpoint

**Symptom**

A tenant member can read another tenant's invoice by changing the numeric ID.

**Evidence**

```text
GET /api/invoices/1042  (tenant A)  -> 200, invoice for tenant B
GET /api/invoices/1043  (tenant A)  -> 200, invoice for tenant B
```

```ts
// src/modules/invoices/invoice.controller.ts:28
const invoice = await invoiceRepository.findById(params.id);
return invoice;
```

**Finding**

- Severity: Critical
- Confidence: Confirmed
- Impact: Cross-tenant disclosure of invoices and customer data.
- Root cause: Object lookup uses only the client-supplied ID; tenant scope is
  not applied.
- Fix: Query by both `id` and `tenantId` from the authenticated context; return
  `404` for objects outside the tenant; add a regression test with two tenants.
- Defense in depth: Centralize tenant scoping in the repository layer and add a
  lint or test rule for unscoped lookups.
- Verification: The same request returns `404`; authorized access still works.

## Example 2: List endpoint ignores authorization scope

**Symptom**

A user can list invoices from all tenants by omitting the tenant filter.

**Evidence**

```ts
// src/modules/invoices/invoice.service.ts:61
const where = {};
if (query.tenantId) where.tenantId = query.tenantId;
return invoiceRepository.find({ where });
```

**Finding**

- Severity: High
- Confidence: Confirmed
- Impact: Bulk data disclosure across tenants.
- Root cause: Authorization scope is optional and derived from client input.
- Fix: Always derive the tenant scope from the authenticated context; ignore or
  reject client-supplied tenant overrides for non-admin roles; add a test that
  removes the filter and expects only the caller's records.
- Verification: A request with another tenant's ID returns no foreign records.

## Example 3: Secret committed to a config file

**Symptom**

A production API key is present in a repository config file.

**Evidence**

```text
config/production.yml:14  api_key: "REDACTED_EXAMPLE_VALUE"
```

**Finding**

- Severity: High
- Confidence: Confirmed
- Impact: Anyone with repository access can use the key until it is rotated.
- Root cause: Secret committed instead of loaded from a secret manager.
- Fix: Rotate the key, remove it from history, load it from the secret manager,
  and add a pre-commit or CI secret scan.
- Verification: The scan is clean; the application starts with the injected
  secret; the old key is rejected.
- Reporting: Do not paste the real value. Record the location, type, and a
  redacted fingerprint.

## Example 4: SSRF in an image import feature

**Symptom**

The image import endpoint fetches any URL supplied by the user, including
internal addresses.

**Evidence**

```ts
// src/modules/media/import.service.ts:37
const response = await fetch(body.imageUrl);
const buffer = Buffer.from(await response.arrayBuffer());
```

**Finding**

- Severity: High
- Confidence: Likely
- Impact: Access to internal services or cloud metadata; potential data
  exfiltration through error messages or stored files.
- Root cause: No scheme, host, port, or resolved-IP allowlist; redirects are
  followed.
- Fix: Allow only `https`, validate against an allowlist of image hosts, resolve
  and reject private/link-local/metadata ranges, disable redirects or revalidate
  each hop, enforce size and content-type limits, and fetch from an isolated
  egress proxy.
- Verification: Internal and metadata URLs are rejected; an allowed test host
  still works; redirects to private ranges are blocked.
- Testing: Use a local test server you control. Do not probe production
  metadata endpoints.

## Example 5: Over-permissive CORS with credentials

**Symptom**

The API reflects any `Origin` and allows credentials.

**Evidence**

```text
Access-Control-Allow-Origin: https://attacker.example
Access-Control-Allow-Credentials: true
```

**Finding**

- Severity: High
- Confidence: Confirmed
- Impact: A malicious site can make authenticated cross-origin requests and read
  responses.
- Root cause: Origin reflection instead of an explicit allowlist.
- Fix: Use a fixed allowlist per environment; never combine wildcard or
  reflection with credentials; validate `Origin` server-side; add tests for
  allowed and denied origins.
- Verification: An unlisted origin receives no CORS approval; listed origins
  still work.

## Report template

```markdown
# Backend Security Report

- Run ID:
- Date:
- Target:
- Environment:
- Authorization:
- Limitations:

## Threat model

- Assets:
- Actors:
- Entry points:
- Trust boundaries:
- Invariants:

## Findings

### SEC-001: <title>

- Severity:
- Confidence:
- Affected endpoint:
- Location:
- Minimal reproduction:
- Evidence (redacted):
- Impact:
- Root cause:
- Recommended fix:
- Defense in depth:
- Verification:
- Regression test:

## Do not ship

- SEC-...

## Unverified or blocked

-

## Appendix

- Entry-point table:
- Commands:
- Redaction notes:
```
