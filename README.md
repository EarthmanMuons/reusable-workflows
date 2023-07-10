# Reusable Workflows

This repository houses a collection of [reusable workflows][] for GitHub
Actions. Its primary goal is to centralize common workflows used across multiple
projects, streamlining the process of updating actions and sharing improvements
consistently.

[reusable workflows]:
  https://docs.github.com/en/actions/using-workflows/reusing-workflows

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
    uses: EarthmanMuons/reusable-workflows/.github/workflows/spelling.yml@main
```

## Workflows

You'll find the following reusable workflows under the [`.github/workflows/`][]
directory of this repository...

[`.github/workflows/`]: .github/workflows/

---

### detect-changed-files.yml

Detects the files changed by a pull request or recently-pushed commit, using
[paths-filter][]. This is useful to enable conditional execution of other
workflow steps.

[paths-filter]: https://github.com/dorny/paths-filter

#### Permissions

This job requires that the environment's `GITHUB_TOKEN` has `read` access
[permissions][] to the `pull-requests` scope, in order to gather metadata on the
changed files.

[permissions]:
  https://docs.github.com/en/actions/security-guides/automatic-token-authentication#permissions-for-the-github_token

#### Outputs

- For each filter, it sets an output variable named by the filter to the text:
  - `"true"` - if **any** of changed files matches any of filter rules
  - `"false"` - if **none** of changed files matches any of filter rules
- For each filter, it sets an output variable with the name
  `${FILTER_NAME}_files`. It will contain a list of all files matching the
  filter.

| Filter Names        |
| :------------------ |
| `added_or_modified` |
| `github_actions`    |
| `markdown`          |

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

  spelling:
    name: spelling
    needs: detect_changed_files
    if: needs.detect_changed_files.outputs.added_or_modified == 'true'
    uses: EarthmanMuons/reusable-workflows/.github/workflows/spelling.yml@main
    with:
      files: ${{ needs.detect_changed_files.outputs.added_or_modified_files }}
```

---

### github-actions.yml

Lints all defined GitHub Actions workflows, using the static checker
[actionlint][].

[actionlint]: https://github.com/rhysd/actionlint

---

### labeler.yml

Labels pull requests based on the paths of files changed, using [labeler][]. The
job expects that the caller repository already has a `.github/labeler.yml`
configuration file defined (see the upstream documentation for details).

[labeler]: https://github.com/actions/labeler

#### Permissions

This job requires that the environment's `GITHUB_TOKEN` has `write` access
[permissions][] to the `pull-requests` scope, in order apply the labels.

You'll want to trigger this job from the [`pull_request_target`][] event, rather
than the [`pull_request`][] event, to securely allow labeling of pull requests
that originate from forked repositories.

[`pull_request_target`]:
  https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#pull_request_target
[`pull_request`]:
  https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows#pull_request

---

### markdown.yml

Checks the formatting of Markdown files, using [Prettier][].

[Prettier]: https://prettier.io/

#### Inputs

| Name               | Type   | Required | Default     |
| :----------------- | :----- | :------- | :---------- |
| `files`            | string | false    | `"**/*.md"` |
| `prettier_version` | string | false    | `"latest"`  |

The `files` input specifies a list of files or glob patterns that the job should
check; if not specified, all Markdown files in the repository will be checked.

The `prettier_version` input specifies the [release version][] of the `prettier`
binary to run; if not specified, the latest available version will be used.

[release version]: https://github.com/prettier/prettier/releases

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
failure in the dependency chain. The job's commit status context can then be
used as the singular required status check when defining branch protection
rules.

#### Inputs

| Name            | Type   | Required |
| :-------------- | :----- | :------- |
| `needs_context` | string | true     |

The [`needs` context][] contains outputs from all jobs that are defined as a
direct dependency of the current job, and is automatically populated by GitHub
Actions. The value should be the JSON representation of that built-in context
information, which you can set using the [expression][] `${{ toJson(needs) }}`.

[`needs` context]:
  https://docs.github.com/en/actions/learn-github-actions/contexts#needs-context
[expression]:
  https://docs.github.com/en/actions/learn-github-actions/expressions

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
      - github_actions
      - markdown
      - spelling
    if: always()
    uses: EarthmanMuons/reusable-workflows/.github/workflows/ready-to-merge.yml@main
    with:
      needs_context: ${{ toJson(needs) }}
```

---

### spelling.yml

Checks for common misspellings in all file types, using [typos][].

[typos]: https://github.com/crate-ci/typos

#### Inputs

| Name    | Type   | Required | Default |
| :------ | :----- | :------- | :------ |
| `files` | string | false    | `"."`   |

The `files` input specifies a list of files or glob patterns that the job should
check; if not specified, all files in the repository will be checked.

---

## License

These reusable workflows are provided under the terms of the [MIT License][].

Copyright &copy; 2023 [Aaron Bull Schaefer](mailto:aaron@elasticdog.com)

[MIT License]: LICENSE
