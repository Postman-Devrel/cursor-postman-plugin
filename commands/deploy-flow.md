---
description: Deploy a Postman Flow so it becomes triggerable, proposing and confirming a trigger path.
allowed-tools: Bash, Read
---

# /postman:deploy-flow -- Deploy a Flow

Deploy a Postman Flow with the Postman CLI so it becomes triggerable. Deploy is mutating — always confirm before running it.

## Prerequisites

This command drives the Postman CLI (`postman flows ...`). If the CLI isn't installed, see https://learning.postman.com/docs/postman-cli/postman-cli-installation/. The CLI authenticates with the same `POSTMAN_API_KEY` this plugin uses — if it's not logged in, run `postman login --with-api-key $POSTMAN_API_KEY`. Reuse the existing CLI session; never authenticate twice.

## Inputs (from the user's message)

- The flow (a 24-character ID, or a name to resolve via `/postman:list-flows`)
- Optionally a desired trigger path and whether auth is required

## Workflow

### Step 1: Resolve the Flow ID

Use `/postman:list-flows` if given a name; ask for the workspace if unknown.

### Step 2: Propose and Confirm

Propose a trigger path derived from the flow name (e.g. "Checkout" → `/checkout`) and **confirm the path and the deploy action** with the user. Deploy is mutating and MUST NOT run without explicit confirmation.

### Step 3: Deploy

Show the command, then run it after confirmation:

```bash
POSTMAN_CLI_SOURCE=cursor-plugin postman flows deploy <flowId> --path /checkout
```

### Step 4: Report

Report the **Trigger URL** and whether the **trigger is enabled**. If it's off, offer `postman flows update <flowId> --trigger on` (confirm first).

### Step 5: Hand Off

If this was part of a deploy-then-trigger request, hand back to `/postman:trigger-flow` to run it.

## Error Handling

| Error | Response |
|-------|----------|
| Postman CLI not installed | "This command needs the Postman CLI. Install it from https://learning.postman.com/docs/postman-cli/postman-cli-installation/ and try again." |
| Flow not found | "I couldn't find that flow. Run /postman:list-flows to see what's in the workspace." |
| Deploy not confirmed | "Deploy is a mutating action — I'll wait for your explicit go-ahead on the trigger path before deploying." |
| Auth failure | "Postman returned 401. Run `postman login --with-api-key $POSTMAN_API_KEY`, or /postman:setup to reconfigure." |

## Related Commands

- Resolve a flow name to an ID -> `/postman:list-flows`
- Run the deployed flow -> `/postman:trigger-flow`
