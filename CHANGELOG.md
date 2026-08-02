# Changelog

All notable changes to this plugin are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Add your changes under `[Unreleased]` in every PR. The release workflow moves that
section into a dated, versioned entry when a release is cut — see [RELEASING.md](RELEASING.md).

## [Unreleased]

## [1.2.0] - 2026-08-02

### Added

- Nine commands to reach the canonical **18-command** set shared with the [Claude Code](https://github.com/Postman-Devrel/postman-claude-code-plugin) and Antigravity plugins: `/postman:run-collection`, `/postman:send-request`, `/postman:list-flows`, `/postman:trigger-flow`, `/postman:deploy-flow`, `/postman:get-flow-run`, `/postman:use-local`, and `/postman:use-remote`. The Flow, `run-collection`, and `send-request` commands drive the Postman CLI; `use-local` / `use-remote` toggle `.mcp.json` between the local stdio package and the hosted server (both API-key authenticated).
- `/postman:generate-client` — generate typed client code **from** a Postman collection. This preserves the client-code-generation capability of the pre-rename `codegen` command (distinct from `/postman:generate-spec`, which generates an OpenAPI spec **from** your code). MCP-tool-driven; requires Code or Full MCP mode.

### Changed

- Renamed `/postman:codegen` to `/postman:generate-spec` and reworked it to generate or update an OpenAPI 3.0 spec from the codebase, matching the canonical command shared across Postman's AI coding plugins. The original client-code-generation capability is preserved as the distinct `/postman:generate-client` command (see Added). Updated the README, routing skill, and knowledge skill references accordingly.
- Updated README installation instructions: Postman is now an official [Cursor Marketplace](https://cursor.com/marketplace/postman) plugin.

## [1.1.0] - 2026-07-20

### Added

- `/postman:learn` command for answering Postman how-to questions.

### Changed

- Reworked the search tool, search strategy, and skills to use private network search.
- Updated README with the proper MCP toolset configuration.

## [1.0.1] - 2026-04-17

### Fixed

- MCP OAuth / authentication flow.
- Corrected the MCP server URL and README repository URL.

## [1.0.0] - 2026-03-09

### Added

- Initial release of the Postman Plugin for Cursor.
- 8 commands covering the full API lifecycle (codegen, docs, mock, search, security, setup, sync, test).
- 3 auto-loaded skills (agent-ready APIs, Postman knowledge, Postman routing).
- `readiness-analyzer` sub-agent (48 checks across 8 pillars).
- API design rules injected into every session.
- Zero-config MCP setup via the Postman MCP Server.

[1.0.1]: https://github.com/Postman-Devrel/cursor-postman-plugin/compare/1.0.0...1.0.1
[1.0.0]: https://github.com/Postman-Devrel/cursor-postman-plugin/releases/tag/1.0.0
[Unreleased]: https://github.com/Postman-Devrel/cursor-postman-plugin/compare/1.2.0...HEAD
[1.2.0]: https://github.com/Postman-Devrel/cursor-postman-plugin/compare/1.1.0...1.2.0
[1.1.0]: https://github.com/Postman-Devrel/cursor-postman-plugin/compare/1.0.1...1.1.0
