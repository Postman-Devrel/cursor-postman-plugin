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

**For private workspace content (user's own APIs):**
1. Call `getWorkspaces` to list workspaces, filter by name if the user mentions one
2. Call `getCollections` to list all collections in the workspace
3. Call `getCollection` (full model) for collections that match the query
4. Scan folder names, request names, and descriptions for the user's query terms

**For public API discovery:**
1. Call `searchPostmanElementsInPublicNetwork` with the user's query
2. Call `getTaggedEntities` to find collections by tag

**Important:** `searchPostmanElementsInPublicNetwork` only searches the Postman public API network. It will NOT find the user's private collections. Always try the private workspace path first.

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

  Want me to generate a client for this API? (/postman:codegen)
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

  Want details on any of these? Or /postman:codegen to generate a client.
```

## Error Handling

| Error | Response |
|-------|----------|
| No results found | "I didn't find any APIs matching that query across your workspaces. Try different keywords, or check if the collection exists in a different workspace." |
| Empty workspace | "This workspace has no collections. Use /postman:sync to create one from an API spec." |
| Auth failure | "Postman returned 401. Your API key may be expired. Run /postman:setup to reconfigure." |
