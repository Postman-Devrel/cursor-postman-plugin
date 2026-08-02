---
description: Discover APIs across your Postman workspaces. Ask natural language questions about available endpoints and capabilities.
allowed-tools: Read, Glob, Grep, mcp__postman__*
---

# /postman:search -- Discover APIs

Answer natural language questions about available APIs. Search across your Postman workspaces to find endpoints, understand capabilities, and drill into details.

## Prerequisites

Postman MCP Server must be configured. If MCP tools fail, tell the user to run `/postman:setup`.

## Workflow

### Step 1: Search

Use the unified `searchPostmanElements` tool. It can search across various entity types like requests, collections, workspaces, specs, flows, environments and mocks.. Choose `entityType`, `ownership`, and `filters` based on the user's intent.

1. Call `searchPostmanElements` with the user's query. Pick the parameters from the user's intent:
   - `entityType`: `requests` (default), `collections`, `workspaces`, `specs`, or `flows`.
   - `ownership`: `organization` (default — your org's resources), `external` (public Postman network, third-party APIs), or `all` (both).
   - `filters`: Optional structured `$and` expression to narrow results — e.g., restrict to the Private API Network, a workspace, or HTTP method.
2. If results are sparse, broaden the search — widen `ownership` to `all`, drop or relax filters, or try a different `entityType`. You can also fall back to `getWorkspaces` + `getCollections` for browsing, or `getTaggedEntities` to find collections by tag.

**Filter examples:**

- Search only the trusted Private API Network: `ownership: organization` with `filters: {"$and":[{"privateNetwork":{"$eq":true}}]}`
- Find a third-party public API (e.g. "Stripe API"): `ownership: external` with `filters: {"$and":[{"visibility":{"$eq":"public"}}]}`
- Restrict to a specific workspace: `filters: {"$and":[{"workspaceId":{"$eq":"ws-abc123"}}]}`
- GET requests only: `entityType: requests` with `filters: {"$and":[{"method":{"$eq":"GET"}}]}`

### Step 2: Drill Into Results

For each relevant match:
1. Call `getCollection` to get the overview
2. Scan endpoint names and descriptions for relevance
3. Call `getCollectionRequest` for the most relevant endpoints to get full details
4. Call `getCollectionResponse` to show what data is returned

### Step 3: Present Results

Format results as a clear answer to the user's question. Answer first, source second.

**When the answer is found:**
```
Yes, you can get a user's email via the API.

  Endpoint: GET /users/{id}
  Collection: "User Management API"
  Auth: Bearer token required

  Response includes:
    {
      "id": "usr_123",
      "email": "jane@example.com",
      "name": "Jane Smith",
      "role": "admin",
      "created_at": "2026-01-15T10:30:00Z"
    }

  Want me to draft an OpenAPI spec from your API code? (/postman:generate-spec)
```

**When the answer is not found:**
```
I didn't find an endpoint that returns user emails.

  Closest matches:
  - GET /users/{id}/profile -- returns name, avatar, but no email
  - GET /users -- list endpoint, doesn't include email in summary view

  The email field might be behind a different permission scope,
  or it might not be exposed via API yet.
```

**When listing capabilities:**
```
Found 3 collections related to "payments":

  1. Payment Gateway API (22 requests)
     - POST /charges -- Create a charge
     - GET /charges/{id} -- Get charge details
     - POST /refunds -- Issue a refund
     ... and 19 more

  2. Subscription API (14 requests)
     - POST /subscriptions -- Create subscription
     - GET /invoices -- List invoices
     ... and 12 more

  3. Billing Admin API (8 requests)
     - GET /revenue/summary -- Revenue dashboard
     ... and 7 more

  Want details on any of these? Or /postman:generate-spec to draft an OpenAPI spec from your code.
```

## Error Handling

| Error | Response |
|-------|----------|
| No results found | "Nothing matched your query. Try different keywords, broaden `ownership` to `all`, or browse workspace collections with `getWorkspaces` + `getCollections`." |
| Empty workspace | "This workspace has no collections. Use /postman:sync to create one from an API spec." |
| Auth failure | "Postman returned 401. Your API key may be expired. Run /postman:setup to reconfigure." |
