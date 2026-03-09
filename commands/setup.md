---
description: Set up Postman MCP Server. Configure API key, verify connection, select workspace.
allowed-tools: mcp__postman__*
---

# /postman:setup -- First-Run Configuration

Guide the user through connecting Cursor to their Postman account via the MCP Server.

## Flow

### Step 1: Check API Key

Check if the `POSTMAN_API_KEY` environment variable is set by calling `getAuthenticatedUser`.

**If the call succeeds:** The API key is valid. Greet the user by name (from the response) and move to Step 2.

**If the call fails or is not configured:**

Tell the user:

```
Postman needs an API key to connect. Here's how to set it up:

1. Go to https://postman.postman.co/settings/me/api-keys
2. Click "Generate API Key"
3. Name it "Cursor Plugin" and copy the key

Then set it as an environment variable:

  export POSTMAN_API_KEY=PMAK-your-key-here

Add it to your shell profile (~/.zshrc or ~/.bashrc) to persist across sessions.

After setting it, restart Cursor and run /postman:setup again.
```

Stop here. Do not proceed until the API key works.

### Step 2: List Workspaces

Call `getWorkspaces` to list all accessible workspaces. Present them:

```
Connected as: Jane Smith (jane@example.com)

Your workspaces:
  1. My Workspace (personal) -- 12 collections
  2. Team API Platform (team) -- 34 collections
  3. Public Examples (public) -- 5 collections
```

Ask which workspace they primarily work in, or note that commands will search across all workspaces by default.

### Step 3: Smoke Test

Pick the workspace the user selected (or the first personal workspace) and call `getCollections` to list collections.

Present a summary:

```
Workspace "My Workspace" has 12 collections:
  - Pet Store API (15 requests)
  - User Management (8 requests)
  - Payment Gateway (22 requests)
  ... and 9 more
```

### Step 4: Confirm Ready

```
You're all set. Postman is connected and ready.

Try these commands:
  /postman:search   -- Find APIs across your workspaces
  /postman:sync     -- Create or update collections from your code
  /postman:codegen  -- Generate typed client code from a collection
  /postman:test     -- Run collection tests
  /postman:mock     -- Create a mock server
  /postman:docs     -- Analyze and improve API documentation
  /postman:security -- Security audit against OWASP API Top 10

Or just describe what you need and I'll route you to the right command.
```

## Error Handling

| Error | Response |
|-------|----------|
| 401 Unauthorized | "Your API key is invalid or expired. Generate a new one at https://postman.postman.co/settings/me/api-keys" |
| 403 Forbidden | "Your API key doesn't have access to this resource. Check your key permissions in Postman settings." |
| Network error | "Can't reach the Postman MCP server. Check your internet connection and try again." |
| No workspaces found | "No workspaces found. This is unusual. Make sure your API key has the right scopes." |
| No collections found | "This workspace is empty. Use /postman:sync to create your first collection from an API spec." |
