---
description: List Postman Flows in a workspace and resolve a flow name to its 24-character ID.
allowed-tools: Bash, Read
---

# /postman:list-flows -- List Flows

List Postman Flows in a workspace with the Postman CLI, and resolve a flow name to its 24-character ID.

## Prerequisites

This command drives the Postman CLI (`postman flows ...`). If the CLI isn't installed, see https://learning.postman.com/docs/postman-cli/postman-cli-installation/. The CLI authenticates with the same `POSTMAN_API_KEY` this plugin uses — if it's not logged in, run `postman login --with-api-key $POSTMAN_API_KEY`. Reuse the existing CLI session; never authenticate twice.

## Inputs (from the user's message)

- The workspace ID (ask if unknown)
- Optionally a name/pattern to filter by

## Workflow

### Step 1: Ensure a Workspace ID

If you don't have one, ask which workspace to list Flows from.

### Step 2: List Flows

```bash
POSTMAN_CLI_SOURCE=cursor-plugin postman flows list --workspace <workspaceId>
```

Narrow with `--filter "<name>"` when resolving a specific flow; use `--sort name` / `--paginate` as needed. Always show the exact command before running it.

### Step 3: Report

Report flow **names + IDs** (and recent status where shown). When resolving a name for another action, return the single matching ID; on multiple matches, present the candidates and ask the user to choose — never guess.

## Error Handling

| Error | Response |
|-------|----------|
| Postman CLI not installed | "This command needs the Postman CLI. Install it from https://learning.postman.com/docs/postman-cli/postman-cli-installation/ and try again." |
| No workspace ID | "Which workspace should I list Flows from? I need its ID." |
| No flows found | "No Flows found in that workspace. Double-check the workspace ID." |
| Auth failure | "Postman returned 401. Run `postman login --with-api-key $POSTMAN_API_KEY`, or /postman:setup to reconfigure." |

## Related Commands

- Trigger a resolved flow -> `/postman:trigger-flow`
- Deploy a flow so it becomes triggerable -> `/postman:deploy-flow`
