---
description: Run a Postman collection with the Postman CLI to verify your API endpoints, then parse and report results.
allowed-tools: Bash, Read, Glob, Grep
---

# /postman:run-collection -- Run a Collection

Run a Postman collection with the Postman CLI to verify your API endpoints. Parse the results, and diagnose failures.

## Prerequisites

This command drives the Postman CLI (`postman collection run`). If the CLI isn't installed, see https://learning.postman.com/docs/postman-cli/postman-cli-installation/. The CLI authenticates with the same `POSTMAN_API_KEY` this plugin uses — if it's not logged in, run `postman login --with-api-key $POSTMAN_API_KEY`.

## Workflow

### Step 1: Find Collections and IDs

List local collection folders and look up their cloud IDs:

```bash
ls postman/collections/
cat .postman/resources.yaml
```

The `cloudResources.collections` section maps local collection paths to cloud IDs.

- If no collections are found, tell the user and stop.
- If there is one collection, use it directly.
- If there are multiple, list them and ask which to run.

### Step 2: Run the Collection

Run by **collection ID** (from `.postman/resources.yaml`):

```bash
postman collection run <collection-id>
```

Common options:

```bash
# Stop on first failure
postman collection run <collection-id> --bail

# With request timeout
postman collection run <collection-id> --timeout-request 10000

# With an environment
postman collection run <collection-id> -e ./postman/environments/<env-file>.json

# Override an environment variable
postman collection run <collection-id> --env-var "base_url=http://localhost:3000"
```

Always show the exact command before running it.

### Step 3: Parse and Report Results

Parse the CLI output for pass/fail counts, failed test names, error messages, and status codes.

### Step 4: Handle Failures

If tests fail:

1. Analyze the error messages.
2. Read the relevant source code.
3. Suggest fixes.
4. After fixes are applied, re-run to verify.

## Error Handling

| Error | Response |
|-------|----------|
| Postman CLI not installed | "This command needs the Postman CLI. Install it from https://learning.postman.com/docs/postman-cli/postman-cli-installation/ and try again." |
| No collections found | "I didn't find any collections under `postman/collections/`. Run /postman:sync to create one, or /postman:search to find one in Postman." |
| Run failed to start | "The collection run failed to start. Check that the collection has at least one request with a valid URL." |
| Auth failure | "Postman returned 401. Your API key may be expired — run `postman login --with-api-key $POSTMAN_API_KEY`, or /postman:setup to reconfigure." |

## Related Commands

- Diagnose failures against your Postman collection with MCP tools -> `/postman:test`
- Send a single ad-hoc request -> `/postman:send-request`
