---
description: Sync Postman collections with your API code. Create collections from specs, push updates, keep everything in sync.
allowed-tools: Bash, Read, Write, Glob, Grep, mcp__postman__*
---

# /postman:sync -- Create/Update Collections

Keep Postman collections in sync with local API code. Detect OpenAPI specs in the project, create new collections, or update existing ones when specs change.

## Prerequisites

Postman MCP Server must be configured. If MCP tools fail, tell the user to run `/postman:setup`.

## Workflow

### Step 1: Understand What Changed

Ask or detect:
- Did the user update an OpenAPI spec? Search for `**/openapi.{json,yaml,yml}`, `**/swagger.{json,yaml,yml}`
- Did they add, remove, or modify endpoints?
- Is there an existing Postman collection to update, or do they need a new one?

If the user provides a specific file path, use that. Otherwise, scan the project for specs.

### Step 2: Find or Create the Collection

**If updating an existing collection:**
1. Call `getCollections` to list collections in the workspace
2. Match by name or ask the user which collection
3. Call `getCollection` to get the current state

**If creating a new collection (decomposed approach, required):**

The `createCollection` MCP tool cannot nest folders. You MUST use this decomposed approach:

1. Read the local OpenAPI spec and parse all endpoints, tags, and schemas
2. Call `createCollection` with the collection name, description, and a `baseUrl` variable. Include one placeholder request since items are required.
3. Call `createCollectionFolder` for each tag or resource group. Parallelize all folder creation calls.
4. Call `createCollectionRequest` for each endpoint, using the `folderId` from step 3. Include:
   - Correct HTTP method and URL with `{{baseUrl}}` prefix
   - `Content-Type: application/json` header on POST/PUT/PATCH requests
   - Realistic JSON request bodies generated from the spec schemas
   - Query parameters (pagination, filters, domain-specific)
   - Path variables using `:variable_name` format
   Batch requests in groups of 25-30 to avoid overwhelming the server.
5. Call `createEnvironment` with variables extracted from the spec:
   - `baseUrl` from the spec's `servers[0].url`
   - Auth variables based on `securitySchemes` (mark as `secret` type)
   - Any common path parameters

### Step 3: Sync

**Spec to Collection (most common):**
1. Call `createSpec` or `updateSpecFile` with the local spec content
2. Call `syncCollectionWithSpec` to update the collection from the spec

**Important:** `syncCollectionWithSpec` returns HTTP 202 (async). Poll `getCollectionUpdatesTasks` until the task completes. Similarly, `generateCollection` is async. Poll `getGeneratedCollectionSpecs` for completion.

**Limitation:** `syncCollectionWithSpec` only supports OpenAPI 3.0. For other versions, use `updateSpecFile` and regenerate the collection.

3. Report what changed: new endpoints, modified schemas, removed paths

**Collection to Spec (reverse sync):**
1. Call `syncSpecWithCollection` to update the spec from collection changes
2. Write the updated spec back to the local file

**Manual updates (no spec):**
For individual endpoint changes:
1. Call `createCollectionRequest` to add new endpoints
2. Call `updateCollectionRequest` to modify existing ones
3. Call `createCollectionFolder` to organize by resource
4. Call `createCollectionResponse` to add example responses

### Step 4: Confirm

Present a clear summary of what changed:

```
Collection synced: "Pet Store API" (15 requests)
  Added:    POST /pets/{id}/vaccinations
  Updated:  GET /pets -- added 'breed' filter parameter
  Removed:  (none)

  Environment: "Pet Store - Development" updated
  Spec Hub: petstore-v3.1.0 pushed
```

## Error Handling

| Error | Response |
|-------|----------|
| No spec files found | "I didn't find any OpenAPI specs in this project. Create one or provide a file path." |
| Multiple specs found | List them and ask which one to sync |
| Collection not found | "No matching collection in Postman. Want me to create a new one from this spec?" |
| Sync task failed | "The sync task failed. Check the spec for validation errors, or try creating the collection fresh." |
| Spec too large (>50KB) | "This spec is too large for direct upload. I'll parse it locally and create collection items directly." |
| Auth failure | "Postman returned 401. Your API key may be expired. Run /postman:setup to reconfigure." |

## Notes

- `createSpec` is impractical for very large specs (>50KB). For large specs, parse locally and create collection items directly using the decomposed approach.
- Always use `getCollection` with the full model to read collections. This returns everything (folders, requests, bodies, params) in a single call.
- `searchPostmanElements` only searches the PUBLIC Postman network. For private workspace search, use `getWorkspaces` + `getCollections`.
