# Reusable Workflows

This repository houses a collection of [reusable workflows][] for GitHub
Actions. Its goal is to centralize CI building blocks used across multiple
projects so they can be updated and improved consistently.

Reusable Workflows is designed to pair with the [Toolbox Envy][] repository.
Where this repository defines high-level CI workflows and execution structure,
Toolbox Envy provides the reusable scripts those workflows invoke, keeping
procedural logic centralized and out of workflow definitions.

[reusable workflows]:
  https://docs.github.com/en/actions/using-workflows/reusing-workflows
[Toolbox Envy]: https://github.com/EarthmanMuons/toolbox-envy

## Usage

You call a reusable workflow by using `uses` at the job level:

- `{owner}/{repo}/.github/workflows/<path>.yml@{ref}`

For example, to call the "spelling.yml" reusable workflow:

```yml
jobs:
  spelling:
    uses: EarthmanMuons/reusable-workflows/.github/workflows/common/check-spelling.yml@main
```

> _For a more comprehensive example of these workflows being used in a
> real-world scenario, see the [WhatChord][] or [spellout][] projects._

[WhatChord]:
  https://github.com/EarthmanMuons/whatchord/tree/main/.github/workflows
[spellout]:
  https://github.com/EarthmanMuons/spellout/tree/main/.github/workflows

## Workflow Index

Reusable workflows are organized by namespace under
[`.github/workflows/`](.github/workflows/).

### Common

General-purpose workflows intended for use by any repository:

- `common/check-github-actions.yml` – Lint workflows via actionlint
- `common/check-markdown.yml` – Format and lint Markdown via Prettier
- `common/check-shell.yml` – Format and lint shell scripts
- `common/check-spelling.yml` – Spellcheck via typos
- `common/detect-changed-files.yml` – Expose paths-filter outputs for
  conditional jobs
- `common/flush-caches.yml` – Delete GitHub Actions caches for the current
  branch
- `common/label-pull-request.yml` – Apply labels via actions/labeler
- `common/preload-caches-actionlint.yml` – Cache the actionlint binary
- `common/ready-to-merge.yml` – Ensure all required jobs truly passed
- `common/tag-if-missing.yml` – Create and push a missing tag using Toolbox Envy
  conventions

Details: [docs/common.md](docs/common.md)

### Flutter

Workflows specific to Flutter projects:

- `flutter/check-flutter.yml` – Format, analyze, and test using pinned Flutter
- `flutter/preload-caches-flutter.yml` – Cache pinned Flutter dependencies
- `flutter/bump-version-flutter.yml` – Bump version and changelog, open PR
- `flutter/draft-release-flutter.yml` – Draft release, build, sign, and upload
  assets

Details: [docs/flutter.md](docs/flutter.md)

### Rust

Workflows specific to Rust projects:

- `rust/check-rust.yml` – fmt, clippy, stable matrix, MSRV
- `rust/check-rust-beta.yml` – Beta toolchain coverage
- `rust/check-rust-miri.yml` – Miri coverage (nightly)
- `rust/preload-caches-rust.yml` – Cache stable and MSRV dependencies
- `rust/bump-version-rust.yml` – cargo-release version and replace, open PR
- `rust/tag-untagged-releases-rust.yml` – Tag and push missing crate tags
- `rust/publish-crate.yml` – Publish crates to crates.io
- `rust/draft-release-rust.yml` – Draft release and upload compiled artifacts
- `rust/deploy-github-pages-rust.yml` – Build rustdoc and deploy to GitHub Pages

Details: [docs/rust.md](docs/rust.md)

### Zig

- `zig/deploy-github-pages-zig.yml` – Build Zig docs and deploy GitHub Pages

Details: [docs/zig.md](docs/zig.md)

## License

These reusable workflows are released under the [Zero Clause BSD
License][LICENSE] (SPDX: 0BSD).

Copyright &copy; 2023 [Aaron Bull Schaefer][EMAIL] and contributors

[LICENSE]: https://github.com/EarthmanMuons/reusable-workflows/blob/main/LICENSE
[EMAIL]: mailto:aaron@elasticdog.com
