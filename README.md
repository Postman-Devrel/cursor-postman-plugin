
# Deprecated
This project has now been deprecated and rolled into the officia [Postman Plugins](https://github.com/postmanlabs/postman-plugin) project. It is now in read-only mode, with no more updates. Please download the plugin directly from the relevant marketplace listings or the github repo linked above.















# Postman Plugin for Cursor

Full API lifecycle management inside Cursor. Sync collections, generate OpenAPI specs and typed clients, discover APIs, run tests, create mocks, run Flows, improve documentation, and audit security. Powered by the [Postman MCP Server](https://github.com/postmanlabs/postman-mcp-server).

> **Note:** This plugin mirrors the [Postman Plugin for Claude Code](https://github.com/Postman-Devrel/postman-claude-code-plugin), which is the source of truth for Postman's AI coding agent plugins.

## What This Plugin Does

This plugin connects Cursor to your Postman account via the Postman MCP Server and bundles purpose-built commands, skills, and an API readiness analyzer. One install gives you:

- **18 commands** covering the complete API lifecycle (the canonical command set shared with the [Claude Code](https://github.com/Postman-Devrel/postman-claude-code-plugin) and Antigravity plugins)
- **3 auto-loaded skills** that teach the agent how to use Postman effectively
- **1 sub-agent** for deep API readiness analysis (48 checks across 8 pillars)
- **API design rules** injected into every session
- **Zero-config MCP setup** (just bring your API key)

## Prerequisites

- [Cursor](https://cursor.com) 2.5+
- A [Postman account](https://www.postman.com) (free tier works)
- A Postman API key

## Installation

### From the Cursor Marketplace (recommended)

Postman is an official [Cursor Marketplace](https://cursor.com/marketplace/postman) plugin.

1. Open Cursor
2. Run `/add-plugin postman`, or browse to the [Postman listing](https://cursor.com/marketplace/postman) and click **Install**
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

### `/postman:generate-spec` -- Generate an OpenAPI Spec

Generate or update an OpenAPI 3.0 spec by scanning the API routes in your codebase, then validate it with the Postman CLI.

```
> /postman:generate-spec
Scanned 8 routes in src/routes/
Wrote postman/specs/openapi.yaml (8 endpoints, 4 schemas)
Validation: passed
```

### `/postman:generate-client` -- Generate Client Code

Generate typed client code from any Postman collection. Detects your project language and matches existing conventions. (The inverse of `generate-spec`: client code **from** a collection, rather than a spec **from** your code.)

```
> /postman:generate-client
Which collection? "User Management API"
Detected: TypeScript project
Generated: src/clients/user-management-api.ts (5 endpoints, 8 types)
```

### `/postman:search` -- Discover APIs

Find APIs across your org's resources, your workspaces, and the public Postman network using natural language questions.

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

### `/postman:learn` -- Learn Postman

Ask "how do I..." questions about Postman itself and how to accomplish workflows. Searches the official Postman documentation and returns cited answers.

```
> /postman:learn how do I create a mock server?
1. Select the collection, open the "..." menu, choose "Mock collection"
2. Name it, optionally link an environment, then create
Source: learning.postman.com/docs/design-apis/mock-apis/...
```

### `/postman:run-collection` -- Run a Collection (CLI)

Run a Postman collection with the Postman CLI, then parse the results and diagnose failures.

```
> /postman:run-collection
Running collection 12345678-...
Passed: 14/15 -- diagnosing the 1 failure...
```

### `/postman:send-request` -- Send an HTTP Request (CLI)

Send a single ad-hoc HTTP request with the Postman CLI and report the response.

```
> /postman:send-request GET https://api.example.com/health
200 OK (82ms) -- {"status":"healthy"}
```

### `/postman:list-flows` -- List Flows

List Postman Flows in a workspace and resolve a flow name to its 24-character ID.

```
> /postman:list-flows
Checkout   -- 6634a1... (last run: passed)
Onboarding -- 6634b2... (last run: failed)
```

### `/postman:trigger-flow` -- Trigger a Flow

Trigger a deployed Postman Flow with inputs from natural language, and report the Run ID, status, and response.

```
> /postman:trigger-flow Checkout with amount 4200
Run ID: run_abc123  Status: 200  Body: {"ok":true}
```

### `/postman:deploy-flow` -- Deploy a Flow

Deploy a Postman Flow so it becomes triggerable, confirming the trigger path first (deploy is mutating).

```
> /postman:deploy-flow Checkout
Proposed path: /checkout -- confirm? (y/n)
Deployed. Trigger URL: https://...  Trigger: enabled
```

### `/postman:get-flow-run` -- Inspect a Flow Run

Inspect a Flow run by its Run ID -- per-block logs, the failing block, and status.

```
> /postman:get-flow-run run_abc123
Status: failed  Failing block: "Charge Card" (502 from upstream)
```

### `/postman:use-local` -- Use the Local MCP Server

Rewrite this plugin's `.mcp.json` to run the Postman MCP server locally over stdio (`npx @postman/postman-mcp-server@latest`), authenticated with your `POSTMAN_API_KEY`.

### `/postman:use-remote` -- Use the Remote MCP Server

Rewrite this plugin's `.mcp.json` back to Postman's hosted MCP server (`https://mcp.postman.com/mcp`), authenticated with your API key.

## Auto-Routing

You don't have to remember command names. The plugin includes a routing skill that maps natural language to the right command:

| You say | Plugin runs |
|---------|------------|
| "Sync my API with Postman" | `/postman:sync` |
| "Generate an OpenAPI spec from my API code" | `/postman:generate-spec` |
| "Generate a Python client for the payments API" | `/postman:generate-client` |
| "Run the checkout collection" | `/postman:run-collection` |
| "Trigger the Checkout flow with amount 4200" | `/postman:trigger-flow` |
| "Run the Postman MCP server locally" | `/postman:use-local` |
| "What endpoints do we have for orders?" | `/postman:search` |
| "Run my API tests" | `/postman:test` |
| "I need a mock for frontend dev" | `/postman:mock` |
| "How do I create a mock server in Postman?" | `/postman:learn` |
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

This plugin ships with the Full (`https://mcp.postman.com/mcp`) endpoint in `.mcp.json`, which covers every MCP-based command including documentation publishing. The Flow, `run-collection`, `send-request`, and `generate-spec` commands drive the Postman CLI instead of MCP, so they work regardless of the MCP mode. Use `/postman:use-remote` or `/postman:use-local` to switch the MCP transport at any time.

**Code mode (default):**
```json
{
  "mcpServers": {
    "postman": {
      "type": "http",
      "url": "https://mcp.postman.com/minimal",
      "headers": {
        "Authorization": "Bearer ${POSTMAN_API_KEY}",
        "X-Source": "cursor-plugin",
        "X-Plugin-Version": "1.2.0",
        "User-Agent": "cursor-postman-plugin/1.2.0"
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
      "url": "https://mcp.postman.com/mcp",
      "headers": {
        "Authorization": "Bearer ${POSTMAN_API_KEY}",
        "X-Source": "cursor-plugin",
        "X-Plugin-Version": "1.2.0",
        "User-Agent": "cursor-postman-plugin/1.2.0"
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
├── commands/                    # 18 commands (canonical cross-plugin set)
│   ├── setup.md                 # /postman:setup
│   ├── sync.md                  # /postman:sync
│   ├── generate-spec.md         # /postman:generate-spec
│   ├── generate-client.md       # /postman:generate-client
│   ├── search.md                # /postman:search
│   ├── test.md                  # /postman:test
│   ├── run-collection.md        # /postman:run-collection
│   ├── send-request.md          # /postman:send-request
│   ├── mock.md                  # /postman:mock
│   ├── docs.md                  # /postman:docs
│   ├── security.md              # /postman:security
│   ├── learn.md                 # /postman:learn
│   ├── list-flows.md            # /postman:list-flows
│   ├── trigger-flow.md          # /postman:trigger-flow
│   ├── deploy-flow.md           # /postman:deploy-flow
│   ├── get-flow-run.md          # /postman:get-flow-run
│   ├── use-local.md             # /postman:use-local
│   └── use-remote.md            # /postman:use-remote
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
4. Add an entry under `[Unreleased]` in [`CHANGELOG.md`](CHANGELOG.md)
5. Test locally: `/add-plugin /path/to/your/fork`
6. Submit a pull request

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

## Versioning & Releases

This plugin follows [Semantic Versioning](https://semver.org/); the current version is in
[`.cursor-plugin/plugin.json`](.cursor-plugin/plugin.json). See [`CHANGELOG.md`](CHANGELOG.md)
for changes between versions and [`RELEASING.md`](RELEASING.md) for the release process.

`/add-plugin Postman-Devrel/cursor-postman-plugin` tracks `main`; releases are tagged (e.g. `1.0.1`)
and published as [GitHub Releases](https://github.com/Postman-Devrel/cursor-postman-plugin/releases).

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
- [Postman on the Cursor Marketplace](https://cursor.com/marketplace/postman)
