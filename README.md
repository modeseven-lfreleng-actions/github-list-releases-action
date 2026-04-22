<!--
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: 2025 The Linux Foundation
-->

# 📦 List GitHub Releases

Returns a list of releases for a GitHub repository. Defaults to the
repository running the workflow, but can target any repository via the
`REPOSITORY` input. Supports text and JSON output formats, tag checking,
and filtering of draft/pre-release entries. Limited to 1000 records due
to GitHub API query limits.

## github-list-releases-action

## Usage Example

Lists repository releases, optionally checking whether a specific tag already
exists as a release.

<!-- markdownlint-disable MD013 -->

```yaml
steps:
  - name: "List GitHub Releases"
    id: list-releases
    uses: lfreleng-actions/github-list-releases-action@main
    with:
      TAG: "v1.0.0"
      SUMMARY_OUTPUT: "true"
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
      REPOSITORY: "lfreleng-actions/test-python-project"
      RETURN_TYPE: "json"
      JSON_FIELDS: "tagName,publishedAt"
```

<!-- markdownlint-enable MD013 -->

## Inputs

<!-- markdownlint-disable MD013 -->

| Name            | Required | Default                    | Description                                    |
| --------------- | -------- | -------------------------- | ---------------------------------------------- |
| TAG             | False    |                            | Check if this tag/version already exists       |
| SUMMARY_OUTPUT  | False    | false                      | Displays summary output in GitHub step summary |
| PRODUCTION_ONLY | False    | false                      | Exclude pre-release and draft releases         |
| RETURN_TYPE     | False    | text                       | Output format: text or json                    |
| JSON_FIELDS     | False    | tagName                    | Fields to return when using JSON return type   |
| ORDER           | False    | desc                       | Sort order: asc or desc                        |
| REPOSITORY      | False    | `${{ github.repository }}` | Target repository in `owner/name` form         |
| GITHUB_TOKEN    | False    | `${{ github.token }}`      | Token used to authenticate the GitHub CLI      |

<!-- markdownlint-enable MD013 -->

## Outputs

<!-- markdownlint-disable MD013 -->

| Name     | Description                                        |
| -------- | -------------------------------------------------- |
| RELEASES | List of releases in specified output format        |
| RELEASED | Whether the provided tag/version already exists    |

<!-- markdownlint-enable MD013 -->

## Requirements/Dependencies

The GitHub CLI (`gh`) must be available in the runner's PATH. This is
pre-installed on GitHub-hosted runners.

The action queries the GitHub API through `gh`, so it does NOT require
a local checkout of the target repository. Set the `REPOSITORY` input
to list releases for any repository.

The action invokes `gh` under the hood, which requires authentication via
the `GH_TOKEN` environment variable. The action sets this automatically
using the `GITHUB_TOKEN` input, which defaults to `${{ github.token }}`.
The calling workflow (or job) must grant at least:

```yaml
permissions:
  contents: read
```

If the default token lacks the required privileges (for example, when
listing releases from a private repository owned by another
organisation), pass an explicit token via the `GITHUB_TOKEN` input.

## Notes

- Setting `RETURN_TYPE` to `json` requires the `JSON_FIELDS` input
  (defaults to `tagName`).
- Tag checking supports both prefixed (`v1.0.0`) and unprefixed (`1.0.0`)
  version strings.
- The action validates the `REPOSITORY` input against the
  `owner/name` pattern; malformed values cause the action to fail fast.
