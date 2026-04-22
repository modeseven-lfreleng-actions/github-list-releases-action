<!--
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2025 The Linux Foundation
-->

# 📦 List GitHub Releases

Returns a list of releases for a GitHub repository. Defaults to the
repository running the workflow, but can target any repository via the
`repository` input. Supports text and JSON output formats, tag
checking, and filtering of draft/pre-release entries. Fetches up to
1000 releases by default, configurable via the `limit` input.

## github-list-releases-action

## Usage Example

Lists repository releases, optionally checking whether a specific tag
already exists as a release.

<!-- markdownlint-disable MD013 -->

```yaml
steps:
  - name: "List GitHub Releases"
    id: list-releases
    uses: lfreleng-actions/github-list-releases-action@main
    with:
      tag: "v1.0.0"
      summary_output: "true"
```

<!-- markdownlint-enable MD013 -->

### Target a Different Repository

The action can query releases for any repository without needing a
checkout step, because the GitHub CLI talks to the API directly.

<!-- markdownlint-disable MD013 -->

```yaml
steps:
  - name: "List releases for another repository"
    uses: lfreleng-actions/github-list-releases-action@main
    with:
      repository: "lfreleng-actions/test-python-project"
      return_type: "json"
      json_fields: "tagName,publishedAt"
```

<!-- markdownlint-enable MD013 -->

## Inputs

<!-- markdownlint-disable MD013 -->

| Name            | Required | Default             | Description                                                       |
| --------------- | -------- | ------------------- | ----------------------------------------------------------------- |
| repository      | False    | current repo        | Target repository in `owner/name` form                            |
| tag             | False    |                     | Check if this tag/version already exists as a release             |
| summary_output  | False    | false               | Write summary output to the GitHub step summary                   |
| production_only | False    | false               | When true, exclude pre-release and draft releases                 |
| return_type     | False    | text                | Output format: `text` or `json`                                   |
| json_fields     | False    | tagName             | Comma-separated fields returned when `return_type` is `json`      |
| order           | False    | desc                | Sort order: `asc` or `desc`                                       |
| limit           | False    | 1000                | Upper bound on releases to fetch (positive integer)               |
| token           | False    | `github.token`      | GitHub token used to authenticate the GitHub CLI                  |

<!-- markdownlint-enable MD013 -->

## Outputs

<!-- markdownlint-disable MD013 -->

| Name     | Description                                                      |
| -------- | ---------------------------------------------------------------- |
| releases | List of releases in the requested output format                  |
| released | Whether the provided tag/version already exists (`true`/`false`) |

<!-- markdownlint-enable MD013 -->

## Requirements/Dependencies

The GitHub CLI (`gh`) must be available in the runner's PATH. This is
pre-installed on GitHub-hosted runners.

The action queries the GitHub API through `gh`, so it does NOT require
a local checkout of the target repository. Set the `repository` input
to list releases for any repository, or omit it to target the workflow
repository (the action defaults `repository` to `${{ github.repository }}`).

The action invokes `gh` under the hood, which requires authentication
via the `GH_TOKEN` environment variable. The action sets this
automatically, using the `token` input when provided and falling back
to `${{ github.token }}` when not. The calling workflow (or job) must
grant at least:

```yaml
permissions:
  contents: read
```

If the default token lacks the required privileges (for example, when
listing releases from a private repository owned by another
organisation), pass an explicit token via the `token` input.

## Notes

- Setting `return_type` to `json` requires a non-empty `json_fields`
  value (defaults to `tagName`). The value must be a comma-separated
  list of alphabetic field names (no empty segments).
- Tag checking supports both prefixed (`v1.0.0`) and unprefixed
  (`1.0.0`) forms: the action checks the provided value first and
  then retries the alternate form when the first pass misses.
- The `released` output always contains `true` or `false`, even
  when callers omit `tag` (in that case, always `false`).
- The action validates all inputs up-front: `repository` must match
  `owner/name`, `return_type` must be `text` or `json`, `order` must
  be `asc` or `desc`, `limit` must be a positive integer, and
  `json_fields` must be alphabetic-with-commas. Malformed values
  cause the action to fail fast with a descriptive error.
- The action truncates release payloads in logs (count plus first
  10 entries for text mode; full suppression for JSON mode). The
  full payload remains available via the `releases` output.
