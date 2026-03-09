---
description: Security audit your APIs against OWASP API Top 10. Finds vulnerabilities and provides remediation guidance.
allowed-tools: Read, Glob, Grep, mcp__postman__*
---

# /postman:security -- API Security Audit

Audit your API for security vulnerabilities. Checks against OWASP API Security Top 10, finds missing auth, exposed sensitive data, insecure transport, weak validation, and more. Works with local OpenAPI specs and Postman collections.

## Prerequisites

Postman MCP Server is optional for this command. Local spec auditing works without MCP. MCP enables collection and environment auditing.

## Workflow

### Step 1: Find the Source

**Local spec:**
- Search for `**/openapi.{json,yaml,yml}`, `**/swagger.{json,yaml,yml}`

**Postman collection (via MCP):**
- Call `getCollections` to list collections
- Call `getCollection` (full model) for complete detail including auth config
- Call `getEnvironment` to check for exposed secrets in environment variables

### Step 2: Run Security Checks

Run all checks and record findings with severity levels.

**Authentication and Authorization:**
- Security schemes defined (OAuth2, API Key, Bearer, etc.)
- Security applied globally or per-endpoint
- No endpoints accidentally unprotected
- OAuth2 scopes defined and appropriate
- Admin endpoints have elevated auth requirements

**Transport Security:**
- All server URLs use HTTPS
- No mixed HTTP/HTTPS endpoints
- HSTS recommended in documentation

**Sensitive Data Exposure:**
- No API keys, tokens, or passwords in example values
- No secrets in query parameters (should be in headers or body)
- Password fields marked as `format: password`
- PII fields identified and noted
- Postman environment variables checked for leaked secrets (via `getEnvironment`)

**Input Validation:**
- All parameters have defined types
- String parameters have `maxLength` (prevents injection and overflow)
- Numeric parameters have `minimum` and `maximum`
- Array parameters have `maxItems`
- Enum values used where applicable
- Request body has required field validation

**Rate Limiting:**
- Rate limits documented in spec
- Rate limit headers defined (`X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset`)
- 429 Too Many Requests response defined

**Error Handling:**
- Error responses don't leak stack traces
- Error schemas don't expose internal field names
- 401 and 403 responses properly defined
- Error messages don't reveal implementation details

**OWASP API Security Top 10:**
- API1: Broken Object Level Authorization (predictable IDs, no ownership checks)
- API2: Broken Authentication (weak auth schemes)
- API3: Broken Object Property Level Authorization (mass assignment risk)
- API4: Unrestricted Resource Consumption (no rate limits, no pagination limits)
- API5: Broken Function Level Authorization (admin endpoints not properly secured)
- API6: Unrestricted Access to Sensitive Business Flows (no abuse prevention)
- API7: Server Side Request Forgery (URL parameters without validation)
- API8: Security Misconfiguration (overly permissive CORS, verbose errors)
- API9: Improper Inventory Management (deprecated endpoints still active)
- API10: Unsafe Consumption of APIs (no validation of third-party responses)

### Step 3: Present Results

```
API Security Audit: pet-store-api.yaml

  CRITICAL (2):
    SEC-001: 3 endpoints have no security scheme applied
      - GET /admin/users
      - DELETE /admin/users/{id}
      - PUT /admin/config
    SEC-002: Server URL uses HTTP (http://api.example.com)

  HIGH (3):
    SEC-003: No rate limiting documentation or 429 response defined
    SEC-004: API key sent as query parameter (use header instead)
    SEC-005: No maxLength on 8 string inputs (injection risk)

  MEDIUM (2):
    SEC-006: Password field visible in GET /users/{id} response
    SEC-007: Environment variable 'db_password' not marked as secret type

  LOW (1):
    SEC-008: No HSTS header recommendation in documentation

  Score: 48/100 -- Significant security issues found
  OWASP Coverage: 6/10 categories have findings
```

### Step 4: Remediation

For each finding, provide:
1. **What's wrong** in plain terms
2. **Why it matters** (the attack vector)
3. **How to fix it** with a specific code example from their spec

Example:
```
SEC-001: Unprotected admin endpoints (CRITICAL)

  These endpoints have no security scheme:
    - GET /admin/users
    - DELETE /admin/users/{id}
    - PUT /admin/config

  Risk: Anyone can access admin functionality without authentication.
  This is OWASP API5 (Broken Function Level Authorization).

  Fix -- add security to each endpoint:

    /admin/users:
      get:
        security:
          - bearerAuth: []
          - apiKey: []
        x-required-role: admin
```

For Postman-specific issues:
- Call `patchEnvironment` to mark secrets properly
- Call `updateCollectionRequest` to fix auth configuration
- Call `updateCollectionResponse` to remove sensitive data from examples

### Step 5: Re-audit

After fixes, offer to re-run the audit and show before/after comparison:

```
Re-audit Results:
  Before: 48/100 (2 critical, 3 high, 2 medium, 1 low)
  After:  82/100 (0 critical, 1 high, 1 medium, 1 low)
  Fixed:  SEC-001, SEC-002, SEC-003, SEC-004, SEC-005, SEC-006
```

## Error Handling

| Error | Response |
|-------|----------|
| No specs or collections found | "I didn't find any API specs or Postman collections to audit. Provide a spec file path or run /postman:sync first." |
| Spec parse error | "Could not parse the OpenAPI spec. Check for YAML/JSON syntax errors." |
| Auth failure | "Postman returned 401. Your API key may be expired. Run /postman:setup to reconfigure." |
