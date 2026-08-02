---
description: Switch this plugin's Postman MCP server to LOCAL (stdio via the npx package), authenticated with your API key.
allowed-tools: Read, Write, Bash
---

# /postman:use-local -- Use the Local MCP Server

Switch the Postman MCP server for this plugin from the remote hosted server to the **local** stdio package (`@postman/postman-mcp-server`, run via `npx`) by rewriting the plugin's own `.mcp.json` in place. Only change the transport (remote → local) — keep the plugin's API-key auth model; this plugin does not use OAuth.

## Workflow

### Step 1: Locate and Read `.mcp.json`

Find this plugin's `.mcp.json` (the file at the plugin root, next to `.cursor-plugin/`) and read its current content so you can confirm the change:

```bash
find "$PWD" -maxdepth 2 -name ".mcp.json" 2>/dev/null; cat ./.mcp.json 2>/dev/null
```

If you can't find it, ask the user for the path to the installed plugin's `.mcp.json`.

### Step 2: Verify the API Key

The local server authenticates with a Postman API key read from the environment (there is no OAuth for the local package):

```bash
[ -n "$POSTMAN_API_KEY" ] && echo "POSTMAN_API_KEY is set" || echo "POSTMAN_API_KEY is NOT set"
```

If it isn't set, tell the user to run `export POSTMAN_API_KEY=<their-key>` and add it to `~/.zshrc` or `~/.bashrc`, then restart the shell. They can get a key at https://postman.postman.co/settings/me/api-keys. Never hardcode the key into the file — it's read from the environment.

### Step 3: Overwrite `.mcp.json`

Write exactly this content to the target `.mcp.json`:

```json
{
  "mcpServers": {
    "postman": {
      "command": "npx",
      "args": ["-y", "@postman/postman-mcp-server@latest"],
      "env": {
        "POSTMAN_API_KEY": "${POSTMAN_API_KEY}"
      }
    }
  }
}
```

`${POSTMAN_API_KEY}` is expanded by Cursor from your environment, and the spawned stdio process also inherits your shell environment. To match Cursor's default remote toolset (the `/mcp` Full endpoint), add `"--full"` to `args`. Use `"--code"` for Code mode or `"--minimal"` for the minimal toolset.

### Step 4: Confirm and Restart

Print the new file content back to the user, then tell them to reload the MCP server in Cursor (**Settings → MCP**, or restart Cursor) for the change to take effect. Remind them they can switch back to the hosted server at any time with `/postman:use-remote`.

## Error Handling

| Error | Response |
|-------|----------|
| `.mcp.json` not found | "I couldn't find the plugin's `.mcp.json`. What's the path to the installed plugin directory?" |
| `POSTMAN_API_KEY` not set | "The local server needs `POSTMAN_API_KEY`. Set it with `export POSTMAN_API_KEY=<key>`, add it to your shell profile, and restart the shell." |
| npx / package fails | "Cursor couldn't start the local server. Make sure Node.js and `npx` are installed and on PATH." |

## Related Commands

- Switch back to the hosted server -> `/postman:use-remote`
