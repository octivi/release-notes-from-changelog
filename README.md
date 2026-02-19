# Extract Release Notes from Changelog GitHub Action

[![GitHub Marketplace](https://img.shields.io/badge/GitHub%20Marketplace-blue?logo=github)](https://github.com/marketplace/actions/release-notes-from-changelog)
[![GitHub Releases](https://img.shields.io/github/v/release/octivi/release-notes-from-changelog?sort=semver)](https://github.com/octivi/release-notes-from-changelog/releases)
[![License: MIT](https://img.shields.io/github/license/octivi/release-notes-from-changelog)](https://choosealicense.com/licenses/mit/)
[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org/)
[![Semantic Versioning](https://img.shields.io/badge/SemVer-2.0.0-blue)](https://semver.org/spec/v2.0.0.html)

This GitHub Action extracts one release section for a given tag from `CHANGELOG.md`.

## What the project does

For a provided tag (for example `v1.2.3`) it finds the matching changelog section and writes that section to a target file (or prints to stdout in standalone mode without `--release-notes`).

## Why the project is useful

It keeps release publishing workflows simple and deterministic when your release body should come directly from `CHANGELOG.md`.

## Why choose this action

- Pure Bash implementation with minimal dependencies
- Fast startup: no `npm`/`pip` install step during workflow execution
- Lower supply-chain and maintenance overhead: no runtime pinning, lockfiles, or dependency CVEs
- Easier security audit: all logic lives in a small, readable script
- Covered by automated tests (`./tests/run`) and CI
- Works on `ubuntu-slim`, which can help reduce runner costs:
  <https://docs.github.com/en/actions/reference/runners/github-hosted-runners>
- Can be used both as a GitHub Action and as a standalone script
- Released under the MIT License: a short and simple permissive license
- Documented security policy: [SECURITY.md](./SECURITY.md)

## Getting started

1. Add the action to your workflow (see "Example workflow" below).
2. Pass the release `tag`.
3. Optionally change `changelog` and `release_notes`.
4. Use the generated file (`release_notes`) as release body in the next step.

## Inputs

| Name            | Required | Default            | Description                                  |
| --------------- | -------- | ------------------ | -------------------------------------------- |
| `tag`           | Yes      | -                  | Release tag to extract, e.g. `v1.2.3`        |
| `changelog`     | No       | `CHANGELOG.md`     | Path to changelog file                       |
| `release_notes` | No       | `release-notes.md` | Output file path for extracted release notes |

## Outputs

| Name            | Description                                                                      |
| --------------- | -------------------------------------------------------------------------------- |
| `release_notes` | Path to file with extracted release notes (same value as `inputs.release_notes`) |

## CLI usage

Run locally from this repo:

```bash
./release-notes-from-changelog \
  --tag "v1.2.3" \
  --changelog "CHANGELOG.md" \
  --release-notes "release-notes.md"
```

To print extracted section to stdout, omit `--release-notes`:

```bash
./release-notes-from-changelog \
  --tag "v1.2.3" \
  --changelog "CHANGELOG.md"
```

Options map 1:1 to the action inputs.

You can also set them via env vars:
`TAG`, `CHANGELOG`, `RELEASE_NOTES`.

If both env vars and CLI options are provided, CLI options take precedence.

## Examples

Accepted changelog headings for tag `v1.2.3`:

```text
## [1.2.3] - 2026-02-19
## [v1.2.3] - 2026-02-19
## 1.2.3 - 2026-02-19
## v1.2.3 - 2026-02-19
```

Example action usage:

```yml
- uses: octivi/release-notes-from-changelog@v1
  with:
    tag: ${{ github.ref_name }}
    changelog: CHANGELOG.md
    release_notes: release-notes.md
```

## Tests

Run the lightweight test suite (no external deps):

```bash
./tests/run
```

## Limitations

- Expects release headings that start with `## ` and end with date in `YYYY-MM-DD`
- Fails if no matching section exists for the tag
- Fails if multiple matching sections exist for the same tag
- Requires exactly one release section per tag in the changelog
- `release_notes`/`--release-notes` must point to a file path (not a directory)
- Output directory for `release_notes` must already exist
- `release_notes`/`--release-notes` must not point to the same file as `changelog`

## Example workflow

```yml
name: Release

on:
  push:
    tags:
      - "v*.*.*"

permissions:
  contents: write

jobs:
  release:
    runs-on: ubuntu-slim
    steps:
      - uses: actions/checkout@v6

      - id: release_notes
        uses: octivi/release-notes-from-changelog@v1
        with:
          tag: ${{ github.ref_name }}
          changelog: CHANGELOG.md
          release_notes: release-notes.md

      - name: Publish release
        uses: softprops/action-gh-release@v2
        with:
          tag_name: ${{ github.ref_name }}
          body_path: ${{ steps.release_notes.outputs.release_notes }}
          generate_release_notes: false
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Getting help

If you run into issues, open a GitHub issue in this repository and include a minimal reproduction
(sample changelog + workflow snippet).

## Other Octivi GitHub Actions

If you are interested in other GitHub Actions we build, see:

- [`octivi/update-copyright-year`](https://github.com/octivi/update-copyright-year) - Updates the copyright year in file headers across your repository
- [`octivi/update-securitytxt-expires`](https://github.com/octivi/update-securitytxt-expires) - Updates the `Expires` field in `security.txt` files to a future date so published security contact metadata stays current
- [`octivi/cloudflare-cache-purge`](https://github.com/octivi/cloudflare-cache-purge) - Purges Cloudflare cache via Cloudflare API

## Maintainers and contributors

Maintained by the [Octivi DevOps team](https://octivi.com/devops). Contributions are welcome via
pull requests.

Built with [Octivi Bash Boilerplate](https://github.com/octivi/bash-boilerplate).
