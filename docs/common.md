# Common Workflows

These workflows are ecosystem-agnostic and intended to be reused across any
repository. They provide CI hygiene, linting, labeling, change detection, cache
maintenance, and release-safety primitives.

## Usage Example

```yml
jobs:
  spelling:
    uses: EarthmanMuons/reusable-workflows/.github/workflows/check-spelling.yml@main
```

---

## detect-changed-files.yml

Detects files changed by a pull request (or recent push) using
[paths-filter](https://github.com/dorny/paths-filter) and exposes the results as
reusable-workflow outputs. This enables conditional execution of downstream jobs
without duplicating filter logic.

**Required permissions**

- `pull-requests: read`

**Outputs**

For each filter name listed below:

- `<filter>` is set to `'true'` if **any** changed file matches the filter
  rules, otherwise `'false'`.
- `<filter>_files` contains a newline-delimited list of matching files (shell
  list format as emitted by paths-filter when `list-files: shell` is enabled).

| Filter              | Meaning                                     |
| ------------------- | ------------------------------------------- |
| `added_or_modified` | Any file added or modified                  |
| `flutter`           | Dart / pubspec / analysis options changes   |
| `github_actions`    | Workflow changes under `.github/workflows/` |
| `markdown`          | Markdown changes                            |
| `rust`              | Rust / Cargo changes                        |
| `zig`               | Zig / zon changes                           |

**Typical usage**

```yml
jobs:
  detect_changed_files:
    permissions:
      pull-requests: read
    uses: EarthmanMuons/reusable-workflows/.github/workflows/detect-changed-files.yml@main

  check_spelling:
    needs: detect_changed_files
    if: needs.detect_changed_files.outputs.added_or_modified == 'true'
    uses: EarthmanMuons/reusable-workflows/.github/workflows/check-spelling.yml@main
    with:
      files: ${{ needs.detect_changed_files.outputs.added_or_modified_files }}
```

---

## check-github-actions.yml

Lints GitHub Actions workflow files using
[actionlint](https://github.com/rhysd/actionlint).

---

## check-markdown.yml

Checks Markdown formatting using [Prettier](https://prettier.io/).

---

## check-shell.yml

Formats and lints shell scripts using
[action-sh-checker](https://github.com/luizm/action-sh-checker).

---

## check-spelling.yml

Detects common misspellings across files using
[typos](https://github.com/crate-ci/typos).

---

## label-pull-request.yml

Applies labels to pull requests based on file paths using
[labeler](https://github.com/actions/labeler).

**Required permissions**

- `pull-requests: write`

**Recommended trigger**

- `pull_request_target`

---

## ready-to-merge.yml

Used as a final "gate" job to ensure the overall workflow outcome matches your
intent when using `needs:` and conditional jobs.

GitHub Actions has an edge case where a failure in an upstream job can cause
dependent jobs to be skipped, and skipped jobs still appear as successful for
the purpose of branch protection unless you explicitly model the failure. This
workflow takes the JSON form of the `needs` context and asserts that every
required job is either:

- `success`, or
- `skipped` (when intentionally conditional)

If any job in the dependency chain is `failure` or `cancelled`, this workflow
fails, allowing you to use a single required status check ("ready to merge") in
branch protection rules.

**Inputs**

| Name            | Required |
| --------------- | -------- |
| `needs_context` | true     |

`needs_context` should be set to `${{ toJson(needs) }}`.

**Typical usage**

The caller should depend on all jobs that must be considered for merge, and it
should always run:

```yml
jobs:
  # ... other jobs ...

  ready_to_merge:
    needs:
      - detect_changed_files
      - check_github_actions
      - check_markdown
      - check_spelling
    if: always()
    uses: EarthmanMuons/reusable-workflows/.github/workflows/ready-to-merge.yml@main
    with:
      needs_context: ${{ toJson(needs) }}
```

---

## preload-caches-actionlint.yml

Caches the latest [actionlint](https://github.com/rhysd/actionlint) binary in
GitHub Actions cache.

---

## flush-caches.yml

Deletes GitHub Actions cache entries for the current branch.

**Required permissions**

- `actions: write`

---

## tag-if-missing.yml

Create and push a missing SemVer tag (via
[Toolbox Envy](https://github.com/EarthmanMuons/toolbox-envy) scripts).

**Inputs**

| Name                | Required |
| ------------------- | -------- |
| `toolbox_envy_bins` | true     |

The `toolbox_envy_bins` entries must exist as `bin/<name>` in Toolbox Envy and
may be newline- or comma-separated; invalid names will fail the job.

**Required secrets**

- `APP_ID`
- `APP_PRIVATE_KEY`

**Authentication**

This workflow uses a GitHub App installation token (via `APP_ID` and
`APP_PRIVATE_KEY`) instead of `GITHUB_TOKEN` so that any tag push reliably
triggers downstream workflows.
