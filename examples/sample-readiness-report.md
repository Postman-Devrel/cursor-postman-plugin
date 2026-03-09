# API Readiness Report: Pet Store API

**Spec:** `openapi/pet-store-v3.yaml`
**Date:** 2026-03-09
**Endpoints:** 15 across 4 resources

---

## Overall Score

```
Score:   67/100
Verdict: NOT AGENT-READY
Reason:  Score below 70% threshold. 2 critical failures.
```

---

## Pillar Breakdown

```
Metadata:        ########..  82%   (6.5/8.0)
Errors:          ####......  41%   (4.5/11.0)
Introspection:   #######...  72%   (7.0/9.5)
Naming:          #########.  91%   (7.5/8.0)
Predictability:  ######....  63%   (5.5/8.5)
Documentation:   ###.......  35%   (2.5/7.0)
Performance:     #####.....  50%   (3.0/6.0)
Discoverability: ########..  80%   (5.5/7.0)
```

---

## Critical Failures (Blockers)

### M1: Missing operationId (3 endpoints)

**Severity:** Critical (4x weight)
**Affected endpoints:**
- `POST /api/v1/pets/{petId}/vaccinations`
- `PUT /api/v1/pets/{petId}/owner`
- `DELETE /api/v1/orders/{orderId}/items/{itemId}`

**Impact:** An agent looking at your 15 endpoints has no reliable way to select these 3. It falls back to URL pattern matching, which is fragile and error-prone.

**Fix:**
```yaml
/api/v1/pets/{petId}/vaccinations:
  post:
    operationId: createPetVaccination    # Add this
    summary: Record a vaccination for a pet

/api/v1/pets/{petId}/owner:
  put:
    operationId: updatePetOwner          # Add this
    summary: Transfer pet ownership

/api/v1/orders/{orderId}/items/{itemId}:
  delete:
    operationId: deleteOrderItem         # Add this
    summary: Remove item from order
```

### E1: Missing error response schemas (13 endpoints)

**Severity:** Critical (4x weight)
**Affected:** 13 of 15 endpoints have no 4xx or 5xx response definitions.

**Impact:** When an agent sends a malformed request and gets a 400 back, it has no idea what the error body looks like. It can't parse the message, identify the bad field, or construct a corrected request.

**Fix:** Add a shared error schema and reference it on all endpoints:
```yaml
components:
  schemas:
    Error:
      type: object
      required: [error, code]
      properties:
        error:
          type: string
          example: "Validation failed"
        code:
          type: string
          example: "VALIDATION_ERROR"
        details:
          type: array
          items:
            type: object
            properties:
              field:
                type: string
                example: "name"
              message:
                type: string
                example: "Name is required"

# Then on each endpoint:
responses:
  '400':
    description: Validation error
    content:
      application/json:
        schema:
          $ref: '#/components/schemas/Error'
  '404':
    description: Resource not found
    content:
      application/json:
        schema:
          $ref: '#/components/schemas/Error'
```

---

## High Priority Fixes

### D2: Rate limits not documented

**Severity:** High (2x weight)
**Impact:** Agent will fire requests as fast as possible until it hits a 429, with no idea when to retry.

**Fix:** Add rate limit info to the API description and define 429 responses:
```yaml
info:
  description: |
    Rate limit: 100 requests per minute per API key.
    Rate limit headers included in all responses.

# On each endpoint:
responses:
  '429':
    description: Rate limit exceeded
    headers:
      Retry-After:
        schema:
          type: integer
        description: Seconds to wait before retrying
      X-RateLimit-Limit:
        schema:
          type: integer
      X-RateLimit-Remaining:
        schema:
          type: integer
```

### P2: Inconsistent pagination

**Severity:** High (2x weight)
**Affected:** GET /pets uses `page`/`pageSize`, GET /orders uses `offset`/`limit`

**Fix:** Standardize on one pattern across all list endpoints. Recommended: `limit`/`offset` with consistent response wrapper.

### I3: Missing enums on constrained fields

**Severity:** High (2x weight)
**Affected:**
- `status` parameter on GET /pets (accepts any string, should be enum)
- `priority` on POST /orders (accepts any string, should be enum)

**Fix:**
```yaml
parameters:
  - name: status
    in: query
    schema:
      type: string
      enum: [available, pending, adopted, fostered]
```

---

## Medium and Low Findings

| Check | Severity | Finding |
|-------|----------|---------|
| M3 | Medium | 5 endpoints missing descriptions |
| I4 | Medium | 18 parameters missing examples |
| I5 | Medium | Date fields missing `format: date-time` |
| P3 | Medium | Mixed date formats (ISO 8601 and Unix timestamps) |
| D3 | Medium | Only 3 of 15 endpoints have response examples |
| PF3 | Medium | No cache headers documented |
| N5 | Low | Mixed camelCase and snake_case in responses |
| DC4 | Low | No contact information in spec |
| DC5 | Low | No license defined |

---

## Estimated Score After Fixes

If you apply the critical and high fixes:

| Fix | Score Impact |
|-----|-------------|
| Add missing operationIds (M1) | +12 |
| Add error response schemas (E1) | +10 |
| Document rate limits (D2) | +4 |
| Standardize pagination (P2) | +4 |
| Add enums to constrained fields (I3) | +3 |

**Projected score: 67 + 33 = ~89/100 (Agent Ready)**

---

## Next Steps

1. Apply the 5 critical and high fixes above (biggest impact)
2. Re-run analysis to verify score improvement
3. Push the improved spec to Postman (`/postman:sync`)
4. Generate a collection and mock server for testing
5. Address medium findings in a follow-up pass
