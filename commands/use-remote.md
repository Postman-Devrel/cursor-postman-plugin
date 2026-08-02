---
description: Switch this plugin's Postman MCP server to the REMOTE hosted server, authenticated with your API key.
allowed-tools: Read, Write, Bash
---

# /postman:use-remote -- Use the Remote MCP Server

Switch the Postman MCP server for this plugin from the local stdio package back to Postman's **remote** hosted server by rewriting the plugin's own `.mcp.json` in place. This plugin authenticates the hosted server with an API-key `Authorization` header (a deliberate workaround for Cursor's OAuth login loop) — do **not** switch it to OAuth.

## Workflow

### Step 1: Locate and Read `.mcp.json`

Find this plugin's `.mcp.json` (the file at the plugin root, next to `.cursor-plugin/`) and read its current content so you can confirm the change:

```bash
find "$PWD" -maxdepth 2 -name ".mcp.json" 2>/dev/null; cat ./.mcp.json 2>/dev/null
```

If you can't find it, ask the user for the path to the installed plugin's `.mcp.json`.

### Step 2: Verify the API Key

The hosted server is reached with an API-key bearer header (no OAuth):

```bash
[ -n "$POSTMAN_API_KEY" ] && echo "POSTMAN_API_KEY is set" || echo "POSTMAN_API_KEY is NOT set"
```

If it isn't set, tell the user to run `export POSTMAN_API_KEY=<their-key>` and add it to `~/.zshrc` or `~/.bashrc`, then restart the shell. They can get a key at https://postman.postman.co/settings/me/api-keys.

### Step 3: Overwrite `.mcp.json`

Write exactly this content to the target `.mcp.json`:

```json
{
  "mcpServers": {
    "postman": {
      "type": "http",
      "url": "https://mcp.postman.com/mcp",
      "headers": {
        "Authorization": "Bearer ${POSTMAN_API_KEY}"
      }
    }
  }
}
```

`${POSTMAN_API_KEY}` is expanded by Cursor from your environment. The `/mcp` path is the Full toolset (Cursor's default for this plugin); use `/code` for Code mode or `/minimal` for the minimal toolset. **EU accounts:** use the `https://mcp.eu.postman.com/...` host instead. Keep the API-key header — do not remove it and rely on OAuth.

### Step 4: Confirm and Restart

Print the new file content back to the user, then tell them to reload the MCP server in Cursor (**Settings → MCP**, or restart Cursor) for the change to take effect. Remind them they can switch to the local package at any time with `/postman:use-local`.

## Error Handling

| Error | Response |
|-------|----------|
| `.mcp.json` not found | "I couldn't find the plugin's `.mcp.json`. What's the path to the installed plugin directory?" |
| `POSTMAN_API_KEY` not set | "The hosted server needs `POSTMAN_API_KEY` for the bearer header. Set it with `export POSTMAN_API_KEY=<key>` and restart the shell." |
| 401 after switching | "The hosted server returned 401. Your API key may be invalid or expired — generate a new one at https://postman.postman.co/settings/me/api-keys." |
| Tool count exceeds Cursor's limit | "The Full (`/mcp`) toolset can exceed Cursor's tool limit. Switch to `/code`, or disable unused tools in Settings → MCP." |

## Related Commands

- Switch to the local stdio package -> `/postman:use-local`
