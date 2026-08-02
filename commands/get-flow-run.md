---
description: Inspect a Postman Flow run by its Run ID — per-block logs, the failing block, and status.
allowed-tools: Bash, Read
---

# /postman:get-flow-run -- Inspect a Flow Run

Inspect a specific Postman Flow run with the Postman CLI. Read-only — no confirmation needed.

## Prerequisites

This command drives the Postman CLI (`postman flows ...`). If the CLI isn't installed, see https://learning.postman.com/docs/postman-cli/postman-cli-installation/. The CLI authenticates with the same `POSTMAN_API_KEY` this plugin uses — if it's not logged in, run `postman login --with-api-key $POSTMAN_API_KEY`. Reuse the existing CLI session; never authenticate twice.

## Inputs (from the user's message)

- The Run ID (the `x-run-id` that `/postman:trigger-flow` reported; ask if unknown)
- Optionally a block ID to focus on

## Workflow

### Step 1: Take the Run ID

If you don't have it, ask for it — it's the `x-run-id` returned when the flow was triggered.

### Step 2: Get the Run

Run a summary first, then add `--logs` for detail:

```bash
POSTMAN_CLI_SOURCE=cursor-plugin postman flows get-run --run-id <runId>
POSTMAN_CLI_SOURCE=cursor-plugin postman flows get-run --run-id <runId> --logs
```

Narrow to a block with `--filter <blockId>`.

### Step 3: Report

Report **which block failed and why**, and the **run status**.

## Error Handling

| Error | Response |
|-------|----------|
| Postman CLI not installed | "This command needs the Postman CLI. Install it from https://learning.postman.com/docs/postman-cli/postman-cli-installation/ and try again." |
| No Run ID | "What's the Run ID? It's the `x-run-id` returned when the flow was triggered." |
| Run not found | "I couldn't find a run with that ID. Double-check the Run ID from the trigger output." |
| Auth failure | "Postman returned 401. Run `postman login --with-api-key $POSTMAN_API_KEY`, or /postman:setup to reconfigure." |

## Related Commands

- Trigger a flow and get a Run ID -> `/postman:trigger-flow`
- List flows in a workspace -> `/postman:list-flows`
