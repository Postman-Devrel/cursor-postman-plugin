# Postman Plugin for Cursor

Full API lifecycle management inside Cursor. Sync collections, generate typed client code, discover APIs, run tests, create mocks, improve documentation, and audit security. Powered by the [Postman MCP Server](https://github.com/postmanlabs/postman-mcp-server).

> **Note:** This plugin mirrors the [Postman Plugin for Claude Code](https://github.com/Postman-Devrel/postman-claude-code-plugin), which is the source of truth for Postman's AI coding agent plugins.

## What This Plugin Does

This plugin connects Cursor to your Postman account via the Postman MCP Server and bundles purpose-built commands, skills, and an API readiness analyzer. One install gives you:

- **8 commands** covering the complete API lifecycle
- **3 auto-loaded skills** that teach the agent how to use Postman effectively
- **1 sub-agent** for deep API readiness analysis (48 checks across 8 pillars)
- **API design rules** injected into every session
- **Zero-config MCP setup** (just bring your API key)

## Prerequisites

- [Cursor](https://cursor.com) 2.5+
- A [Postman account](https://www.postman.com) (free tier works)
- A Postman API key

## Installation

### From the Cursor Marketplace

> **Coming soon:** Cursor Marketplace listing and `/add-plugin` support are not yet available. Use the GitHub or local install methods below for now.

1. Open Cursor
2. Run `/add-plugin postman` or browse the [marketplace](https://cursor.com/marketplace)
3. Set your API key (see Setup below)

### From GitHub

```
/add-plugin Postman-Devrel/cursor-postman-plugin
```

### Local Development

```
/add-plugin /path/to/cursor-postman-plugin
```

## Setup

### 1. Get a Postman API Key

1. Go to [Postman API Keys](https://postman.postman.co/settings/me/api-keys)
2. Click **Generate API Key**
3. Name it "Cursor Plugin" and copy the key (starts with `PMAK-`)

### 2. Set the Environment Variable

Add to your shell profile (`~/.zshrc`, `~/.bashrc`, or `~/.bash_profile`):

```bash
export POSTMAN_API_KEY=PMAK-your-key-here
```

Restart your terminal (or run `source ~/.zshrc`).

### 3. Verify Connection

In Cursor, run:

```
/postman:setup
```

This verifies your API key, lists your workspaces, and confirms everything is connected.

## Commands

### `/postman:setup` -- First-Run Configuration

Guides you through API key setup, verifies the MCP connection, and lists your workspaces.

### `/postman:sync` -- Sync Collections

Create or update Postman collections from your local OpenAPI specs. Keeps your Postman workspace in sync with your code.

```
> /postman:sync
Found openapi.yaml in ./api/openapi.yaml
Creating collection "Pet Store API" with 15 endpoints...
Collection synced. Environment "Pet Store - Dev" created.
```

### `/postman:codegen` -- Generate Client Code

Generate typed client code from any Postman collection. Detects your project language and matches existing conventions.

```
> /postman:codegen
Which collection? "User Management API"
Detected: TypeScript project
Generated: src/clients/user-management-api.ts (5 endpoints, 8 types)
```

### `/postman:search` -- Discover APIs

Find APIs across your org's private network, your workspaces, and the public Postman network using natural language questions.

```
> /postman:search
"Is there an endpoint that returns user email addresses?"
Yes -- GET /users/{id} in the "User Management API" collection returns email.
```

### `/postman:test` -- Run Collection Tests

Execute Postman collection tests, analyze failures, and get fix suggestions.

```
> /postman:test
Running "Pet Store API" tests...
Passed: 12/15 (80%)
Failed: 3 -- diagnosing...
```

### `/postman:mock` -- Create Mock Servers

Create mock servers from your collections for frontend development and testing.

```
> /postman:mock
Created mock: https://abc123.mock.pstmn.io
Add to .env: API_BASE_URL=https://abc123.mock.pstmn.io
```

### `/postman:docs` -- API Documentation

Analyze documentation completeness and generate missing descriptions, examples, and error docs.

```
> /postman:docs
Documentation coverage: 60%
Missing: 12 error responses, 23 parameter descriptions
Want me to fill the gaps?
```

### `/postman:security` -- Security Audit

Audit your API against OWASP API Security Top 10. Finds vulnerabilities and provides specific remediation.

```
> /postman:security
Score: 48/100
CRITICAL: 3 endpoints have no auth
HIGH: No rate limiting defined
Providing fixes...
```

## Auto-Routing

You don't have to remember command names. The plugin includes a routing skill that maps natural language to the right command:

| You say | Plugin runs |
|---------|------------|
| "Sync my API with Postman" | `/postman:sync` |
| "Generate a Python client for the payments API" | `/postman:codegen` |
| "What endpoints do we have for orders?" | `/postman:search` |
| "Run my API tests" | `/postman:test` |
| "I need a mock for frontend dev" | `/postman:mock` |
| "Is my API agent-ready?" | Readiness Analyzer agent |

## API Readiness Analyzer

The plugin includes a sub-agent that evaluates your APIs for AI agent compatibility. It runs 48 checks across 8 pillars and scores your API on a 0-100 scale.

Trigger it with:
- "Is my API agent-ready?"
- "Scan my API for AI compatibility"
- "What's wrong with my API for agents?"

It analyzes your OpenAPI spec, identifies issues, and walks you through fixes. It can push improved specs back to Postman.

See `examples/sample-readiness-report.md` for a sample output.

## Configuration

### MCP Server Modes

This plugin defaults to **Code mode** (~45-50 tools), which covers 7 of 8 commands fully. The only gap is documentation publishing (available in Full mode only).

**Code mode (default):**
```json
{
  "mcpServers": {
    "postman": {
      "type": "http",
      "url": "https://mcp.postman.com/code",
      "headers": {
        "Authorization": "Bearer ${POSTMAN_API_KEY}"
      }
    }
  }
}
```

**Full mode (power users, 100+ tools):**

Edit `.mcp.json` in the plugin directory:
```json
{
  "mcpServers": {
    "postman": {
      "type": "http",
      "url": "https://mcp.postman.com",
      "headers": {
        "Authorization": "Bearer ${POSTMAN_API_KEY}"
      }
    }
  }
}
```

Full mode includes `publishDocumentation` / `unpublishDocumentation` but exceeds Cursor's 80-tool limit. You may need to disable unused tools in Cursor Settings > MCP.

### Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `POSTMAN_API_KEY` | Yes | Your Postman API key (starts with `PMAK-`) |

## Plugin Structure

```
cursor-postman-plugin/
├── .cursor-plugin/
│   └── plugin.json              # Plugin manifest
├── .mcp.json                    # Postman MCP server config (Code mode)
├── commands/
│   ├── setup.md                 # /postman:setup
│   ├── sync.md                  # /postman:sync
│   ├── codegen.md               # /postman:codegen
│   ├── search.md                # /postman:search
│   ├── test.md                  # /postman:test
│   ├── mock.md                  # /postman:mock
│   ├── docs.md                  # /postman:docs
│   └── security.md              # /postman:security
├── skills/
│   ├── postman-routing/         # Auto-routes intent to commands
│   ├── postman-knowledge/       # Postman concepts + MCP guidance
│   └── agent-ready-apis/        # API readiness knowledge
├── agents/
│   └── readiness-analyzer.md    # 48-check API readiness analyzer
├── rules/
│   └── postman-best-practices.mdc  # API design rules
├── assets/                      # Logo and branding
├── examples/
│   └── sample-readiness-report.md
├── LICENSE                      # Apache-2.0
└── README.md
```

## Contributing

1. Fork this repository
2. Create a feature branch: `git checkout -b feature/my-improvement`
3. Make your changes
4. Test locally: `/add-plugin /path/to/your/fork`
5. Submit a pull request

### Adding a New Command

1. Create `commands/your-command.md` with YAML frontmatter
2. Add routing patterns to `skills/postman-routing/SKILL.md`
3. Test with `/postman:your-command` in Cursor
4. Update this README

### Testing Locally

```bash
# Clone the repo
git clone https://github.com/Postman-Devrel/cursor-postman-plugin.git

# Install in Cursor
# In Cursor agent, run:
/add-plugin /path/to/cursor-postman-plugin

# Verify
/postman:setup
```

## License

[Apache-2.0](LICENSE)

## See Also

- [Postman Plugin for Claude Code](https://github.com/Postman-Devrel/postman-claude-code-plugin) - The source of truth for all Postman AI coding plugins
- [Postman Agent Skills](https://github.com/Postman-Devrel/agent-skills) - Portable skills for any skills.sh-compatible agent
- [Postman Cursor Rules](https://github.com/Postman-Devrel/postman-cursor-rules) - Lightweight MCP config + rules for Cursor

## Links

- [Postman MCP Server](https://github.com/postmanlabs/postman-mcp-server)
- [Postman API Documentation](https://learning.postman.com/docs/)
- [Cursor Plugin Documentation](https://cursor.com/docs/plugins)
- [Cursor Marketplace](https://cursor.com/marketplace) *(coming soon)*
