---
description: Analyze, improve, and generate API documentation from Postman collections and specs.
allowed-tools: Read, Glob, Grep, mcp__postman__*
---

# /postman:docs -- API Documentation

Analyze API documentation completeness, generate missing descriptions and examples, and improve documentation quality. Works with local OpenAPI specs and Postman collections.

**Note:** Code mode does not include `publishDocumentation` or `unpublishDocumentation`. This command analyzes and improves docs but does not publish. To publish, use Full mode or publish directly from Postman.

## Prerequisites

Postman MCP Server must be configured. If MCP tools fail, tell the user to run `/postman:setup`.

## Workflow

### Step 1: Find the Source

Check for API definitions in this order:

**Local specs:**
- Search for `**/openapi.{json,yaml,yml}`, `**/swagger.{json,yaml,yml}`

**Postman specs:**
- Call `getAllSpecs` to find specs already in Postman
- Call `getSpecDefinition` to pull the full definition

**Postman collections:**
- Call `getCollections` to find relevant collections
- Call `getCollection` (full model) to get complete collection detail

### Step 2: Analyze Documentation Completeness

Read the spec or collection and assess coverage:

```
Documentation Coverage: Pet Store API

  Overall:                        60%
  Endpoints with descriptions:    8/15  (53%)
  Parameters with descriptions:   22/45 (49%)
  Endpoints with examples:        3/15  (20%)
  Error responses documented:     2/15  (13%)
  Authentication documented:      Yes
  Rate limits documented:         No

  Biggest gaps:
  - 12 endpoints missing error response docs
  - 23 parameters missing descriptions
  - 12 endpoints missing request/response examples
```

### Step 3: Generate or Improve

**If documentation is sparse (< 40% coverage):** Generate documentation for each endpoint:
- Operation summary and description inferred from endpoint name, path, and HTTP method
- Parameter table (name, type, required, description)
- Request body schema with realistic examples
- Response schemas with examples for each status code (200, 201, 400, 404, 500)
- Error response documentation
- Authentication requirements per endpoint

**If documentation is partial (40-80%):** Fill the gaps:
- Add missing descriptions inferred from naming, schemas, and existing patterns
- Generate realistic examples from schemas
- Add error response documentation
- Document authentication and rate limits
- Standardize description format across all endpoints

**If documentation is good (> 80%):** Review and refine:
- Check description quality and consistency
- Verify examples match current schemas
- Suggest improvements for clarity

### Step 4: Apply Changes

Ask the user which output format they want:

1. **Update the spec file** -- Write improved docs back into the OpenAPI spec directly
2. **Update in Postman** -- Use `updateCollectionRequest` to add descriptions, examples, and documentation to each request in the collection
3. **Generate markdown** -- Create a `docs/api-reference.md` file for the project
4. **Show diff** -- Display what would change without applying

### Step 5: Sync Spec and Collection

If both a spec and collection exist, keep them in sync:
- Call `syncCollectionWithSpec` to update collection from spec changes (async, poll for completion)
- Or call `syncSpecWithCollection` to update spec from collection changes

## Error Handling

| Error | Response |
|-------|----------|
| No specs or collections found | "I didn't find any API specs or Postman collections. Create an OpenAPI spec or use /postman:sync to import one." |
| Collection is empty | "This collection has no requests. Add endpoints first, then run /postman:docs to generate documentation." |
| Publish not available | "Documentation publishing requires Full mode. You can publish directly from Postman at https://learning.postman.com/docs/publishing-your-api/publishing-your-docs/" |
| Auth failure | "Postman returned 401. Your API key may be expired. Run /postman:setup to reconfigure." |
