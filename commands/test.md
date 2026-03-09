---
description: Run Postman collection tests, analyze results, diagnose failures, and suggest fixes.
allowed-tools: Bash, Read, Write, Glob, Grep, mcp__postman__*
---

# /postman:test -- Run Collection Tests

Execute Postman collection tests from Cursor. Analyze results, diagnose failures, and suggest code fixes.

## Prerequisites

Postman MCP Server must be configured. If MCP tools fail, tell the user to run `/postman:setup`.

## Workflow

### Step 1: Find the Collection

1. Call `getWorkspaces` to find the target workspace
2. Call `getCollections` to list available collections
3. Match by name or ask the user which collection to test

If the user provides a collection ID directly, skip to Step 2.

### Step 2: Select Environment

If the collection uses environment variables:
1. Call `getEnvironments` to list available environments
2. Ask which environment to use, or detect from naming convention (e.g., "Development", "Staging", "Production")
3. Note the environment ID for the run

### Step 3: Run Tests

Call `runCollection` with:
- The collection UID
- The environment ID (if applicable)

### Step 4: Parse and Present Results

Present test results clearly:

```
Test Results: Pet Store API
  Environment: Development
  Requests:  15 executed
  Passed:    12 (80%)
  Failed:    3

  Failures:
  1. POST /users -> "Status code is 201" -> Got 400
     Request: createUser
     Folder: User Management

  2. GET /users/{id} -> "Response has email field" -> Missing
     Request: getUser
     Folder: User Management

  3. DELETE /users/{id} -> "Status code is 204" -> Got 403
     Request: deleteUser
     Folder: User Management
```

### Step 5: Diagnose Failures

For each failure:
1. Call `getCollectionRequest` to see the full request definition (URL, body, headers, test scripts)
2. Call `getCollectionResponse` to see expected responses
3. Check if the API source code is in the current project
4. Explain what the test expected versus what it received
5. If the source code is local, find the handler and suggest the fix

Present diagnosis:

```
Diagnosis: POST /users returned 400 instead of 201

  The test sends:
    { "name": "Test User", "email": "test@example.com" }

  But looking at your handler (src/routes/users.ts:45),
  the 'role' field is now required since commit abc123.

  Fix: Add "role" to the test request body, or make 'role' optional
  in your validation schema.
```

### Step 6: Fix and Re-run

After the user applies fixes:
1. Offer to re-run: "Want me to run the collection again?"
2. Call `runCollection` again
3. Show before/after comparison:

```
Re-run Results: Pet Store API
  Passed:  14/15 (93%) -- up from 12/15 (80%)
  Fixed:   POST /users, GET /users/{id}
  Still failing: DELETE /users/{id} (403 Forbidden)
```

### Step 7: Update Collection Tests (if needed)

If the tests themselves need updating (not the API code):
- Call `updateCollectionRequest` to fix request bodies, headers, or test scripts
- Call `updateCollectionResponse` to update expected response examples

## Error Handling

| Error | Response |
|-------|----------|
| Collection not found | "I didn't find that collection. Run /postman:search to see available collections." |
| No test scripts | "This collection doesn't have test scripts. Tests are written in the 'Tests' tab of each request in Postman." |
| Environment missing variables | "The environment is missing required variables: [list]. Update them in Postman or provide values." |
| Run failed to start | "The collection run failed to start. Check that the collection has at least one request with a valid URL." |
| Auth failure | "Postman returned 401. Your API key may be expired. Run /postman:setup to reconfigure." |
