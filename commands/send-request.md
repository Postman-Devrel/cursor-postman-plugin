---
description: Send a single HTTP request with the Postman CLI, then report the response.
allowed-tools: Bash, Read, Glob, Grep
---

# /postman:send-request -- Send an HTTP Request

Send an HTTP request with the Postman CLI. Ask the user for the URL and method, or detect them from context.

## Prerequisites

This command drives the Postman CLI (`postman request`). If the CLI isn't installed, see https://learning.postman.com/docs/postman-cli/postman-cli-installation/. The CLI authenticates with the same `POSTMAN_API_KEY` this plugin uses — if it's not logged in, run `postman login --with-api-key $POSTMAN_API_KEY`.

## Workflow

### Step 1: Determine Request Details

Ask the user for:

- The URL to send the request to
- The HTTP method (default: GET)
- Any headers, body, or auth needed

If the user wants to send a request from a collection, find collection folders under `postman/collections/` and read the `*.request.yaml` files to extract the method and URL. Collections use the v3 folder format.

### Step 2: Build and Execute

```bash
postman request <METHOD> "<URL>"
```

- **With headers:** add `-H "Header: value"`
- **With a body:** add `-d '{"key": "value"}'`
- **With bearer auth:** add `--auth-bearer-token "<token>"`
- **With API key:** add `--auth-apikey-key "<name>" --auth-apikey-value "<key>"`
- **With basic auth:** add `--auth-basic-username "<user>" --auth-basic-password "<pass>"`
- **With an environment:** add `-e ./postman/environments/<file>.json`

Always show the exact command before running it.

### Step 3: Report Results

Parse the response and report the status code, response time, and body. Suggest fixes for errors (auth issues, connection problems, invalid URLs).

## Error Handling

| Error | Response |
|-------|----------|
| Postman CLI not installed | "This command needs the Postman CLI. Install it from https://learning.postman.com/docs/postman-cli/postman-cli-installation/ and try again." |
| Missing URL | "What URL should I send the request to, and which HTTP method?" |
| Connection error | "I couldn't reach that URL. Check the host, port, and that the service is running." |
| Auth failure | "The request returned 401/403. Check the auth flags (bearer token, API key, or basic auth)." |

## Related Commands

- Run a full collection's tests -> `/postman:run-collection`
