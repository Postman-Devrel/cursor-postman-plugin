# Postman MCP Server: Known Limitations and Workarounds

These constraints were discovered during real-world testing. Follow them to avoid common failures.

## 1. `searchPostmanElementsInPublicNetwork` is public-only

**Problem:** This tool only searches the Postman public API network. It will NOT find the user's private collections or workspaces.

**Workaround:** For private workspace search, use `getWorkspaces` + `getCollections` + `getCollection`. Scan collection names, request names, and descriptions locally.

**Rule:** Always try the private workspace path first. Only fall back to public search if the user explicitly asks about public APIs or if the private search finds nothing.

## 2. `generateCollection` is async (HTTP 202)

**Problem:** This tool returns immediately with a task ID, not the finished collection.

**Workaround:** Poll `getGeneratedCollectionSpecs` with the task ID until the task status is "completed". Check every 2-3 seconds, timeout after 60 seconds.

## 3. `syncCollectionWithSpec` is async and OpenAPI 3.0 only

**Problem:** Returns HTTP 202 (async). Also only supports OpenAPI 3.0 specs, not 2.0 (Swagger) or 3.1.

**Workaround:**
- Poll `getCollectionUpdatesTasks` for completion status
- For non-3.0 specs: use `updateSpecFile` to push the spec, then regenerate the collection with `generateCollection`

## 4. `createCollection` cannot nest folders

**Problem:** The MCP schema for `createCollection` has `additionalProperties: false` on item entries with no `item` property for sub-items. You cannot create a collection with nested folder structure in one call.

**Workaround:** Use the decomposed approach:
1. `createCollection` with a shell (name, description, one placeholder request)
2. `createCollectionFolder` for each folder (parallelize these calls)
3. `createCollectionRequest` for each endpoint with `folderId` to place in the correct folder
4. Batch request creation in groups of 25-30

## 5. `putCollection` auth enum lacks "noauth"

**Problem:** When updating collection-level auth, the `type` enum doesn't include "noauth" as a valid option.

**Workaround:** Set auth at the request level instead, or inherit from collection-level settings. If you need "no auth", omit the auth configuration entirely.

## 6. `createSpec` is impractical for large specs (>50KB)

**Problem:** Very large OpenAPI specs may fail or timeout when uploaded via `createSpec`.

**Workaround:** Parse the spec locally from the filesystem and create collection items directly using the decomposed approach (createCollection + createCollectionFolder + createCollectionRequest). This is more reliable for large APIs with many endpoints.

## 7. Code mode tool availability

**Problem:** This plugin uses Code mode (~45-50 tools). Some tools available in Full mode (100+ tools) are not available:
- `publishDocumentation` / `unpublishDocumentation`

**Workaround:** For doc publishing, direct the user to publish from the Postman web app. Everything else works in Code mode.

**Power users:** To switch to Full mode, edit `.mcp.json` and change the URL from `https://mcp.postman.com/code` to `https://mcp.postman.com`. Note: Full mode has 100+ tools which may exceed Cursor's 80-tool limit. You may need to disable unused tools in Cursor Settings.

## 8. `getCollection` model parameter

**Tip:** Always call `getCollection` with the full model to get everything (folders, requests, bodies, params) in a single call. Without this, you get a minimal response that requires additional calls.

## 9. Async operation general pattern

For any tool that returns HTTP 202:
1. Capture the task ID from the response
2. Poll the corresponding status endpoint every 2-3 seconds
3. Set a reasonable timeout (60 seconds for most operations)
4. If the task fails, check the error message and suggest a fix
5. If the task times out, suggest the user check the Postman web app for status
