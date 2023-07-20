# Reusable Workflows

This repository houses a collection of [reusable workflows][] for GitHub
Actions. Its primary goal is to centralize common workflows used across multiple
projects, streamlining the process of updating actions and sharing improvements
consistently.

## Usage

In GitHub Actions, you call a reusable workflow by using the `uses` keyword in a
YAML workflow file. Unlike when you are using actions within a workflow, you
call reusable workflows directly within a job, and not from within job steps.
You can reference reusable workflow files using the following syntax:

- `{owner}/{repo}/.github/workflows/{filename}@{ref}`

For example, to call the "spelling.yml" reusable workflow:

```yml
jobs:
  spelling:
    uses: EarthmanMuons/reusable-workflows/.github/workflows/check-spelling.yml@main
```

## Workflows

You'll find the following reusable workflows under the
[`.github/workflows/`](.github/workflows/) directory of this repository...

---

### check-github-actions.yml

Lints all defined GitHub Actions workflows, using the static checker
[actionlint][].

---

### check-markdown.yml

Checks the formatting of Markdown files, using [Prettier][].

#### Inputs

| Name               | Type   | Required | Default   |
| :----------------- | :----- | :------- | :-------- |
| `files`            | string | false    | `**/*.md` |
| `prettier_version` | string | false    | `latest`  |

The `files` input specifies a list of files or glob patterns that the job should
check; if not specified, all Markdown files in the repository will be checked.

The `prettier_version` input specifies the release version of the [`prettier`
binary][] to run; if not specified, the latest available version will be used.

---

### check-rust.yml

Performs comprehensive testing on a Rust project, including checking for proper
formatting with `cargo fmt`, linting with `cargo clippy`, running unit tests
across `macos-latest`, `ubuntu-latest`, and `windows-latest` using the latest
stable release of Rust, and finally testing on the Minimum Supported Rust
Version (MSRV).

---

### check-spelling.yml

Checks for common misspellings in all file types, using [typos][].

#### Inputs

| Name    | Type   | Required | Default |
| :------ | :----- | :------- | :------ |
| `files` | string | false    | `.`     |

The `files` input specifies a list of files or glob patterns that the job should
check; if not specified, all files in the repository will be checked.

---

### detect-changed-files.yml

Detects the files changed by a pull request or recently-pushed commit, using
[paths-filter][]. This is useful to enable conditional execution of other
workflow steps.

#### Permissions

This job requires that the environment's `GITHUB_TOKEN` has `read` access
[permissions][] to the `pull-requests` scope, in order to gather metadata on the
changed files.

#### Outputs

- For each filter, it sets an output variable named by the filter to the text:
  - `'true'` - if **any** of changed files matches any of filter rules
  - `'false'` - if **none** of changed files matches any of filter rules
- For each filter, it sets an output variable with the name
  `${FILTER_NAME}_files`. It will contain a list of all files matching the
  filter.

| Filter Names        |
| :------------------ |
| `added_or_modified` |
| `github_actions`    |
| `markdown`          |
| `rust`              |

#### Example

Only check for common misspellings in files that were added or modified in the
current pull request:

```yml
jobs:
  detect_changed_files:
    name: detect changed files
    permissions:
      pull-requests: read
    uses: EarthmanMuons/reusable-workflows/.github/workflows/detect-changed-files.yml@main

  check_spelling:
    name: check spelling
    needs: detect_changed_files
    if: needs.detect_changed_files.outputs.added_or_modified == 'true'
    uses: EarthmanMuons/reusable-workflows/.github/workflows/check-spelling.yml@main
    with:
      files: ${{ needs.detect_changed_files.outputs.added_or_modified_files }}
```

---

### flush-cache.yml

Deletes all cache entries from the [GitHub Actions cache][] for the current
branch.

#### Permissions

This job requires that the environment's `GITHUB_TOKEN` has `write` access
[permissions][] to the `actions` scope, in order delete entries from the GitHub
Actions cache.

---

### github-pages-rust.yml

Generates the [rustdoc][] documentation for the project's local package(s) and
then deploys it to the repository's [GitHub Pages][] environment.

#### Permissions

This job requires that the environment's `GITHUB_TOKEN` has `write` access
[permissions][] for both the `id-token` and `pages` scopes. The `id-token` scope
is needed to verify the source of the deployment, while the `pages` scope allows
the job to deploy to [GitHub Pages][].

---

### labeler.yml

Labels pull requests based on the paths of files changed, using [labeler][]. The
job expects that the caller repository already has a `.github/labeler.yml`
configuration file defined (see the upstream documentation for details).

#### Permissions

This job requires that the environment's `GITHUB_TOKEN` has `write` access
[permissions][] to the `pull-requests` scope, in order apply the labels.

You'll want to trigger this job from the [`pull_request_target`][] event, rather
than the [`pull_request`][] event, to securely allow labeling of pull requests
that originate from forked repositories.

---

### preload-cache-actionlint.yml

Saves the [actionlint][] binary into the [GitHub Actions cache][].

---

### preload-cache-rust.yml

Saves the repository's Rust project dependencies for both the _stable_ and
_MSRV_ (Minimum Supported Rust Version) toolchains into the [GitHub Actions
cache][], using [rust-cache][].

---

### ready-to-merge.yml

Used as the final step in pull request workflows to ensure that all required
jobs have actually succeeded or been conditionally skipped as expected. When
using job dependencies within GitHub Actions, there's an edge case where the
failure of a parent job will cause any children jobs that depend on it to be
skipped; and skipped jobs are considered to be a _pass_ when they're used for
required status checks.

To prevent false positive cases due to the above behavior, the "ready to merge"
job will explicitly check all of the `needs` job statuses to ensure there was no
failure in the dependency chain. The commit status context of this job can then
be used as the singular required status check when defining branch protection
rules.

#### Inputs

| Name            | Type   | Required |
| :-------------- | :----- | :------- |
| `needs_context` | string | true     |

The [`needs` context][] contains outputs from all jobs that are defined as a
direct dependency of the current job, and is automatically populated by GitHub
Actions. Here, the `needs_context` input value should be the JSON representation
of that built-in context information, which you can set using the [expression][]
`${{ toJson(needs) }}`.

#### Example

The caller of this job should _always_ be run and have an explicit dependency on
all other jobs defined in the workflow that are required to pass:

```yml
jobs:
  # other jobs not shown...

  ready_to_merge:
    name: ready to merge
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

## License

These reusable workflows are provided under the terms of the
[MIT License](LICENSE).

Copyright &copy; 2023 [Aaron Bull Schaefer](mailto:aaron@elasticdog.com)

<!-- REFERENCES -->

[actionlint]: https://github.com/rhysd/actionlint
[expression]:
  https://docs.github.com/en/actions/learn-github-actions/expressions
[GitHub Actions cache]:
  https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows
[GitHub Pages]: https://docs.github.com/pages
[labeler]: https://github.com/actions/labeler
[`needs` context]:
  https://docs.github.com/en/actions/learn-github-actions/contexts#needs-context
[paths-filter]: https://github.com/dorny/paths-filter
[permissions]:
  https://docs.github.com/en/actions/security-guides/automatic-token-authentication#permissions-for-the-github_token
[Prettier]: https://prettier.io/
[`prettier` binary]: https://github.com/prettier/prettier/releases
[`pull_request`]:
  https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#pull_request
[`pull_request_target`]:
  https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#pull_request_target
[reusable workflows]:
  https://docs.github.com/en/actions/using-workflows/reusing-workflows
[rust-cache]: https://github.com/Swatinem/rust-cache
[rustdoc]: https://doc.rust-lang.org/rustdoc/index.html
[typos]: https://github.com/crate-ci/typos
