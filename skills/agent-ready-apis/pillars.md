# The 8 Pillars of Agent-Ready APIs

Complete reference for all 48 checks across 8 pillars. Each check includes what to look for, severity, and why agents care.

---

## Pillar 1: Metadata (6 checks)

Agents need rich metadata to discover and select the right endpoint.

| # | Check | Severity | What to Look For |
|---|-------|----------|-----------------|
| M1 | operationId on every endpoint | Critical | Every path+method has a unique operationId |
| M2 | Summary on every endpoint | High | Short human-readable summary (< 120 chars) |
| M3 | Description on every endpoint | Medium | Detailed description of what the endpoint does |
| M4 | Tags on every endpoint | Medium | At least one tag for grouping/categorization |
| M5 | Consistent tag naming | Low | Tags use consistent casing and naming pattern |
| M6 | x-display-name or similar metadata | Low | Enhanced display metadata for tools |

**Why agents care:** Without operationIds, an agent looking at 50 endpoints has no reliable way to select the right one. It falls back to URL pattern matching, which breaks on non-standard paths.

---

## Pillar 2: Errors (7 checks)

Agents need structured errors to self-heal.

| # | Check | Severity | What to Look For |
|---|-------|----------|-----------------|
| E1 | Error response schemas defined | Critical | 4xx and 5xx responses have schemas |
| E2 | Consistent error format | High | All errors follow the same schema structure |
| E3 | Error codes defined | High | Machine-readable error codes (not just HTTP status) |
| E4 | Error messages present | Medium | Human-readable error messages in schema |
| E5 | Retry guidance (Retry-After) | Medium | 429 and 503 responses include retry-after info |
| E6 | Validation error details | Medium | 400 responses include field-level validation errors |
| E7 | No stack traces in examples | Low | Error examples don't leak internal details |

**Why agents care:** When an agent gets a 400 error with no schema, it can't parse the error message to figure out what went wrong. It either retries blindly or gives up. With structured errors, the agent can read the error code, identify the bad field, fix the request, and retry.

---

## Pillar 3: Introspection (7 checks)

Agents need parameter details to construct valid requests.

| # | Check | Severity | What to Look For |
|---|-------|----------|-----------------|
| I1 | Parameter types defined | Critical | All parameters have explicit types (string, integer, etc.) |
| I2 | Required fields marked | High | Required parameters/properties explicitly marked |
| I3 | Enum values for constrained fields | High | Fields with limited valid values use enum |
| I4 | Examples on parameters | Medium | Parameters have example values |
| I5 | Format specifiers | Medium | Dates use format: date-time, emails use format: email |
| I6 | Default values documented | Low | Optional params with defaults have them documented |
| I7 | Nullable fields marked | Low | Fields that can be null are explicitly marked nullable |

**Why agents care:** Without explicit types and enums, an agent constructing a request has to guess. Should "status" be "active", "ACTIVE", "1", or true? Enums eliminate this guessing.

---

## Pillar 4: Naming (6 checks)

Agents need predictable patterns to reason about APIs.

| # | Check | Severity | What to Look For |
|---|-------|----------|-----------------|
| N1 | RESTful URL patterns | High | Resources use nouns (/users, /orders), not verbs |
| N2 | Consistent path casing | High | All paths use the same casing (kebab-case preferred) |
| N3 | HTTP method semantics | Medium | GET reads, POST creates, PUT replaces, PATCH updates, DELETE removes |
| N4 | Plural resource names | Medium | Collections use plural (/users not /user) |
| N5 | Consistent property casing | Low | Response properties use consistent casing (camelCase or snake_case) |
| N6 | No action verbs in URLs | Low | Use HTTP methods instead of /getUser or /deleteOrder |

**Why agents care:** Predictable naming lets agents infer URLs. If /users exists, an agent can predict /users/{id} exists. If naming is inconsistent (/getUsers, /order_delete), the agent can't make these inferences.

---

## Pillar 5: Predictability (6 checks)

Agents need consistent response structures.

| # | Check | Severity | What to Look For |
|---|-------|----------|-----------------|
| P1 | Response schemas defined | Critical | All success responses have schemas |
| P2 | Consistent pagination | High | List endpoints use consistent pagination pattern |
| P3 | Consistent date formats | Medium | All dates use ISO 8601 format |
| P4 | Consistent ID formats | Medium | All resource IDs use the same format (UUID, integer, prefixed) |
| P5 | Envelope vs. direct responses | Low | Consistent choice of wrapper objects vs. direct arrays |
| P6 | Consistent null handling | Low | Null fields handled consistently (omitted vs. explicit null) |

**Why agents care:** If GET /users returns `{ "data": [...], "meta": {...} }` but GET /orders returns a bare array, the agent needs different parsing logic for each. Consistency lets agents write one parser.

---

## Pillar 6: Documentation (6 checks)

Agents need context that humans get from reading docs.

| # | Check | Severity | What to Look For |
|---|-------|----------|-----------------|
| D1 | Authentication documented | High | Security schemes fully described with flow details |
| D2 | Rate limits documented | High | Rate limit policies described (requests per minute, etc.) |
| D3 | Request/response examples | Medium | At least one example per endpoint |
| D4 | External documentation links | Low | Links to guides, tutorials, or detailed docs |
| D5 | Deprecation notices | Low | Deprecated endpoints marked with alternatives |
| D6 | Changelog or versioning | Low | API versioning strategy documented |

**Why agents care:** An agent can't "read the docs page" the way a human can. All context must be embedded in the spec. If rate limits aren't in the spec, the agent will learn about them the hard way (by getting 429'd).

---

## Pillar 7: Performance (5 checks)

Agents need to operate within constraints.

| # | Check | Severity | What to Look For |
|---|-------|----------|-----------------|
| PF1 | Rate limit headers documented | High | X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset |
| PF2 | Pagination on list endpoints | High | List endpoints support pagination (limit/offset or cursor) |
| PF3 | Cache headers documented | Medium | ETag, Cache-Control, Last-Modified headers |
| PF4 | Bulk/batch endpoints | Low | Batch operations for reducing request count |
| PF5 | Async patterns for long operations | Low | Long-running operations return 202 with status polling |

**Why agents care:** An agent that doesn't know about rate limits will fire requests as fast as possible until it gets blocked. An agent without pagination will try to load entire datasets in one call, causing timeouts or memory issues.

---

## Pillar 8: Discoverability (5 checks)

Agents need to find and connect to the API.

| # | Check | Severity | What to Look For |
|---|-------|----------|-----------------|
| DC1 | Valid OpenAPI version | High | OpenAPI 3.0+ (3.1 preferred) |
| DC2 | Server URLs defined | High | At least one server URL (production) |
| DC3 | API info complete | Medium | Title, version, description in info block |
| DC4 | Contact information | Low | Contact email or URL for support |
| DC5 | License defined | Low | License field populated (MIT, Apache-2.0, etc.) |

**Why agents care:** Server URLs tell the agent where to send requests. Without them, the agent has to guess or ask the user. OpenAPI version affects which features the agent can rely on.

---

## Scoring Formula

```
max_score = sum(severity_weight * check_count) for all checks
actual_score = sum(severity_weight) for all passing checks
percentage = (actual_score / max_score) * 100

Severity weights:
  Critical = 4
  High = 2
  Medium = 1
  Low = 0.5
```

**Total checks:** 48
**Max weighted score:** (5 * 4) + (12 * 2) + (12 * 1) + (19 * 0.5) = 20 + 24 + 12 + 9.5 = 65.5

**Agent Ready threshold:** >= 70% (45.85+ weighted score) AND zero critical failures.
