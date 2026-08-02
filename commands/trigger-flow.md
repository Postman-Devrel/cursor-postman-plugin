---
description: Trigger (run) a deployed Postman Flow with inputs, and deploy-then-trigger it if it isn't deployed yet.
allowed-tools: Bash, Read
---

# /postman:trigger-flow -- Trigger a Flow

Trigger a deployed Postman Flow from natural language, using the Postman CLI.

## Prerequisites

This command drives the Postman CLI (`postman flows ...`). If the CLI isn't installed, see https://learning.postman.com/docs/postman-cli/postman-cli-installation/. The CLI authenticates with the same `POSTMAN_API_KEY` this plugin uses — if it's not logged in, run `postman login --with-api-key $POSTMAN_API_KEY`. Reuse the existing CLI session; never authenticate twice.

## Inputs (from the user's message)

- The flow (a 24-character ID, or a name to resolve via `/postman:list-flows`)
- Any inputs / query params / headers / scenario
- The workspace ID (ask if a name needs resolving and you don't have it)

## Workflow

### Step 1: Resolve the Flow ID

Use `/postman:list-flows` if given a name; disambiguate multiple matches; ask for the workspace if unknown.

### Step 2: Build the Flags

Translate natural language into flags: `-i k=v` (inputs), `-q k=v` (query params), `--headers k=v`, `-s "<scenario>"`.

### Step 3: Trigger

Show the command, then run it:

```bash
POSTMAN_CLI_SOURCE=cursor-plugin postman flows trigger <flowId> -i amount=4200
```

### Step 4: Report

Report the **Run ID**, **HTTP status**, and **response body**.

### Step 5: Handle Edge Cases

- If the flow **is not deployed** → explain and offer to deploy it (via `/postman:deploy-flow`, explicit confirmation required), then re-trigger.
- If the trigger is **disabled** → offer `postman flows update <flowId> --trigger on` (confirm first), then trigger.
- On a **non-2xx** response → surface the status + body and offer `/postman:get-flow-run --run-id <id>` for per-block detail.

Confirm before any mutating action (deploy, enable trigger).

## Error Handling

| Error | Response |
|-------|----------|
| Postman CLI not installed | "This command needs the Postman CLI. Install it from https://learning.postman.com/docs/postman-cli/postman-cli-installation/ and try again." |
| Flow not found | "I couldn't find that flow. Run /postman:list-flows to see what's in the workspace." |
| Flow not deployed | "That flow isn't deployed yet. Want me to deploy it first with /postman:deploy-flow?" |
| Auth failure | "Postman returned 401. Run `postman login --with-api-key $POSTMAN_API_KEY`, or /postman:setup to reconfigure." |

## Related Commands

- Resolve a flow name to an ID -> `/postman:list-flows`
- Deploy a flow first -> `/postman:deploy-flow`
- Inspect a failed run -> `/postman:get-flow-run`
