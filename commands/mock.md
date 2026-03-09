---
description: Create Postman mock servers for frontend development. Generates missing examples, provides integration config.
allowed-tools: Bash, Read, Write, Glob, Grep, mcp__postman__*
---

# /postman:mock -- Create Mock Servers

Create Postman mock servers from your collections or API specs. Get a working mock URL for frontend development, integration testing, or demos.

## Prerequisites

Postman MCP Server must be configured. If MCP tools fail, tell the user to run `/postman:setup`.

## Workflow

### Step 1: Find the Source

**Option A: From existing collection**
1. Call `getCollections` to list available collections
2. Select the target collection
3. Call `getCollection` (full model) to check its contents

**Option B: From local spec**
1. Find OpenAPI spec in the project (`**/openapi.{json,yaml,yml}`, `**/swagger.*`)
2. Import it first:
   - Call `createSpec` with the spec content
   - Call `generateCollection` from the spec (async, poll `getGeneratedCollectionSpecs` for completion)

### Step 2: Check for Response Examples

Mock servers serve example responses. Check if the collection has saved response examples.

Call `getCollection` and inspect requests for saved responses.

If examples are missing:

```
Your collection doesn't have response examples. Mock servers
need these to know what to return.

I'll generate realistic examples from your schemas and add them
to the collection.
```

For each request without examples:
1. Call `getCollectionRequest` to get the request definition and any schema info
2. Generate a realistic example response based on the schema, endpoint name, and HTTP method
3. Call `createCollectionResponse` to save the example with:
   - Appropriate status code (200 for GET, 201 for POST, 204 for DELETE)
   - Realistic response body with sample data
   - Correct Content-Type header

### Step 3: Create Mock Server

Call `createMock` with:
- `collectionId`: The collection to mock
- `name`: `<api-name> Mock`
- `environment`: Environment ID if applicable
- `isPublic`: false by default (private mock requires Postman API key header)

### Step 4: Present Mock URL

```
Mock server created: "Pet Store API Mock"
  URL: https://<mock-id>.mock.pstmn.io
  Status: Active
  Visibility: Private (requires x-api-key header)

  Try it:
    curl -H "x-api-key: $POSTMAN_API_KEY" https://<mock-id>.mock.pstmn.io/pets
    curl -H "x-api-key: $POSTMAN_API_KEY" https://<mock-id>.mock.pstmn.io/pets/1

  The mock serves the example responses from your collection.
  Update examples in Postman to change mock behavior.
```

### Step 5: Integration Snippets

Provide code to integrate the mock into the user's project:

```
Quick integration:

  # Add to your .env file
  API_BASE_URL=https://<mock-id>.mock.pstmn.io
  POSTMAN_MOCK_API_KEY=<your-postman-api-key>

  # In your code
  const API_URL = process.env.API_BASE_URL || 'https://<mock-id>.mock.pstmn.io';
```

### Step 6: Publish (optional)

If the user wants the mock publicly accessible (no API key required):
- Call `publishMock` to make it publicly available
- Useful for demos, hackathons, or public documentation

To revert to private:
- Call `unpublishMock`

## Error Handling

| Error | Response |
|-------|----------|
| Collection not found | "I didn't find that collection. Run /postman:search to see available collections." |
| No examples to mock | "This collection has no response examples and no schemas to generate from. Add example responses in Postman first." |
| Mock creation failed | "Mock server creation failed. Make sure the collection has at least one request with a saved response example." |
| Auth failure | "Postman returned 401. Your API key may be expired. Run /postman:setup to reconfigure." |
