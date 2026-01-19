# Rust Workflows

Reusable workflows for [Rust](https://rust-lang.org/) projects including
linting, multi-platform testing, release preparation, tagging, publishing,
caching, and documentation deployment.

> **Note on authentication**
>
> Some workflows in this namespace require a GitHub App installation token
> instead of the default `GITHUB_TOKEN`.
>
> This is intentional. Commits, pull requests, and tags created using
> `GITHUB_TOKEN` do not always trigger subsequent workflows due to GitHub’s
> event-propagation restrictions. Using a GitHub App token ensures that version
> bumps, release preparation, and automated tagging behave exactly as if they
> were performed by a normal user, allowing downstream workflows to run
> reliably.
>
> Any workflow that requires `APP_ID` and `APP_PRIVATE_KEY` is using this
> mechanism.

## Usage Example

```yml
jobs:
  rust_ci:
    uses: EarthmanMuons/reusable-workflows/.github/workflows/rust/check-rust.yml@main
```

---

## check-rust.yml

Runs formatting, clippy linting, stable multi-platform tests, and MSRV tests.

---

## check-rust-beta.yml

Runs the full Rust test suite against the latest beta toolchain.

---

## check-rust-miri.yml

Runs tests using nightly Rust with the
[Miri interpreter](https://github.com/rust-lang/miri).

---

## preload-caches-rust.yml

Caches Rust dependencies for both stable and MSRV toolchains.

---

## bump-version-rust.yml

Uses [cargo-release](https://github.com/crate-ci/cargo-release) to bump
versions, update documentation, and open a PR.

**Inputs**

- package
- level

**Secrets**

- `APP_ID`
- `APP_PRIVATE_KEY`

**Authentication**

This workflow requires a GitHub App token (see namespace note above).

---

## tag-untagged-releases-rust.yml

Adds missing crate version tags using cargo-release and pushes them.

**Required secrets**

- `APP_ID`
- `APP_PRIVATE_KEY`

**Authentication**

This workflow requires a GitHub App token (see namespace note above).

---

## publish-crate.yml

Publishes an unpublished Rust crate to crates.io.

**Required secret**

- `CARGO_REGISTRY_TOKEN`

---

## draft-release-rust.yml

Creates a GitHub draft release and uploads compiled artifacts across platforms.

---

## deploy-github-pages-rust.yml

Builds rustdoc and deploys documentation to GitHub Pages.

**Required permissions**

- `pages: write`
- `id-token: write`
