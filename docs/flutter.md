# Flutter Workflows

Reusable workflows for [Flutter](https://flutter.dev/) applications, covering CI
validation, dependency caching, version bumping, and release automation for
signed Android artifacts.

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
  flutter_ci:
    uses: EarthmanMuons/reusable-workflows/.github/workflows/flutter/check-flutter.yml@main
```

---

## check-flutter.yml

Runs formatting, static analysis, and tests for Flutter projects using the
Flutter version pinned in `pubspec.yaml`.

Steps include:

- `dart format --set-exit-if-changed`
- `flutter analyze`
- `flutter test`

---

## preload-caches-flutter.yml

Preloads Flutter dependencies for the pinned Flutter SDK into GitHub Actions
cache to speed up CI runs.

---

## bump-version-flutter.yml

Bumps the project semantic version and CHANGELOG and opens a release PR.

**Inputs**

| Name  | Required |
| ----- | -------- |
| level | true     |

**Secrets**

- `APP_ID`
- `APP_PRIVATE_KEY`

**Authentication**

This workflow requires a GitHub App token (see namespace note above).

---

## draft-release-flutter.yml

Creates a draft GitHub release, builds signed Android artifacts, stages them,
and uploads them to the release.

**Inputs**

| Name              | Required | Default |
| ----------------- | -------- | ------- |
| slug              | true     | —       |
| android_key_alias | false    | release |

**Secrets**

- `ANDROID_KEYSTORE_B64`
- `ANDROID_KEYSTORE_PASSWORD`
- `ANDROID_KEY_PASSWORD`
