# Changelog

All notable changes to this plugin are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Add your changes under `[Unreleased]` in every PR. The release workflow moves that
section into a dated, versioned entry when a release is cut — see [RELEASING.md](RELEASING.md).

## [Unreleased]

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
[Unreleased]: https://github.com/Postman-Devrel/cursor-postman-plugin/compare/1.1.0...HEAD
[1.1.0]: https://github.com/Postman-Devrel/cursor-postman-plugin/compare/1.0.1...1.1.0
