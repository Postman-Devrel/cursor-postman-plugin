# Releasing

This plugin is distributed straight from GitHub (`/add-plugin Postman-Devrel/cursor-postman-plugin`).
A release is a git tag plus a GitHub Release — there is no build or package step. Cursor
installs the repo at a ref, so the tag *is* the artifact.

## Versioning

We follow [Semantic Versioning](https://semver.org/): `MAJOR.MINOR.PATCH`, tagged with the
bare version (no `v` prefix). The current version lives in
[`.cursor-plugin/plugin.json`](.cursor-plugin/plugin.json).

| Bump  | When |
| ----- | ---- |
| MAJOR | Breaking changes: renamed/removed commands or skills, or MCP changes that break existing installs. |
| MINOR | New command, skill, or agent, or an additive capability. |
| PATCH | Fixes, doc/README tweaks, prompt refinements with no new surface. |

## Day-to-day: every PR

When your PR changes plugin content (`commands/`, `skills/`, `agents/`, `rules/`, `.mcp.json`),
add an entry under `## [Unreleased]` in [`CHANGELOG.md`](CHANGELOG.md), grouped as
`Added` / `Changed` / `Fixed` / `Removed`. **Do not bump `plugin.json` yourself** — the release
workflow does that. The `PR Check` workflow fails if you change content without a changelog entry.

## Cutting a release

1. Make sure `[Unreleased]` in `CHANGELOG.md` reflects everything on `main` to be shipped.
2. Go to **Actions → Release → Run workflow**.
3. Pick the `bump` (`patch` / `minor` / `major`) and run.

The workflow then:

- bumps `version` in `.cursor-plugin/plugin.json`,
- renames `## [Unreleased]` to `## [X.Y.Z] - <date>` and opens a fresh empty `[Unreleased]`,
- commits `chore(release): X.Y.Z` to `main`,
- creates and pushes the annotated tag `X.Y.Z`,
- publishes a GitHub Release whose notes are that version's changelog section.

It fails fast if `[Unreleased]` is empty or the target tag already exists.
