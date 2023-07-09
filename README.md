# Reusable Workflows

A collection of [reusable workflows][] for GitHub Actions.

[reusable workflows]:
  https://docs.github.com/en/actions/using-workflows/reusing-workflows

## Usage

You call a reusable workflow by using the `uses` keyword. Unlike when you are
using actions within a workflow, you call reusable workflows directly within a
job, and not from within job steps. You can reference reusable workflow files
using the following syntax:

- `{owner}/{repo}/.github/workflows/{filename}@{ref}`

For example, to call the "spelling.yml" reusable workflow:

```yml
jobs:
  spelling:
    uses: EarthmanMuons/reusable-workflows/.github/workflows/spelling.yml@main
```

### detect-changed-files.yml

Detects the files changed by a pull request or recently-pushed commit, using
[paths-filter][]. This is useful to enable conditional execution of other
workflow steps.

[paths-filter]: https://github.com/dorny/paths-filter

#### Outputs

- For each filter, it sets an output variable named by the filter to the text:
  - `"true"` - if **any** of changed files matches any of filter rules
  - `"false"` - if **none** of changed files matches any of filter rules
- For each filter, it sets an output variable with the name
  `${FILTER_NAME}_files`. It will contain a list of all files matching the
  filter.

| Defined Filter Names |
| -------------------- |
| `added_or_modified`  |
| `github_actions`     |
| `markdown`           |

### github-actions.yml

Lints all defined GitHub Actions workflows, using the static checker
[actionlint][].

[actionlint]: https://github.com/rhysd/actionlint

### labeler.yml

Labels pull requests based on the paths of files changed, using [labeler][]. The
job expects that the caller repository already has a `.github/labeler.yml`
configuration file defined (see the upstream documentation for details).

[labeler]: https://github.com/actions/labeler

### markdown.yml

Checks the formatting of Markdown files, using [prettier][].

[prettier]: https://prettier.io/

#### Inputs

| Name               | Type   | Required | Default     |
| ------------------ | ------ | -------- | ----------- |
| `files`            | string | false    | `"**/*.md"` |
| `prettier_version` | string | false    | `"3.0.0"`   |

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
| --------------- | ------ | -------- |
| `needs_context` | string | true     |

#### Example

The caller of this job should _always_ be run and have an explicit dependency on
all other required jobs defined in the workflow:

```yml
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

### spelling.yml

Checks for common misspellings in all file types, using [typos][].

[typos]: https://github.com/crate-ci/typos

#### Inputs

| Name    | Type   | Required | Default |
| ------- | ------ | -------- | ------- |
| `files` | string | false    | `"."`   |

## License

These workflows are provided under the terms of the
[MIT License](https://en.wikipedia.org/wiki/MIT_License).

Copyright &copy; 2023 [Aaron Bull Schaefer](mailto:aaron@elasticdog.com)
