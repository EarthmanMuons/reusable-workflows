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
    uses: EarthmanMuons/reusable-workflows/.github/workflows/check-flutter.yml@main
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

## bump-version-flutter-app.yml

Intended for Flutter applications released as final artifacts (APK/IPA/etc).

Bumps the project version and CHANGELOG (using CalVer), then opens a release
preparation pull request.

**Inputs**

| Name          | Required |
| ------------- | -------- |
| date_override | false    |
| format        | false    |

**Secrets**

- `APP_ID`
- `APP_PRIVATE_KEY`

**Authentication**

This workflow requires a GitHub App token (see namespace note above).

---

## bump-version-flutter-lib.yml

Intended for Flutter packages/plugins consumed as dependencies.

Bumps the project version and CHANGELOG (using SemVer), then opens a release
preparation pull request.

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

| Name                          | Required | Default    |
| ----------------------------- | -------- | ---------- |
| slug                          | true     | —          |
| build_android_aab             | false    | true       |
| build_ios_ipa                 | false    | true       |
| android_app_signing_key_alias | false    | appsigning |
| android_upload_key_alias      | false    | upload     |

**Environment Configuration**

This workflow reads signing material from the `signing` environment in the
calling repository. Callers do not need to pass signing secrets through
`workflow_call`.

**Required in `signing` environment (always)**

- `secrets.ANDROID_APP_SIGNING_KEYSTORE_B64`: Base64-encoded Android app-signing
  keystore (PKCS12 or JKS) used by Gradle when building release APKs.
- `secrets.ANDROID_APP_SIGNING_KEYSTORE_PASSWORD`: Password protecting the
  Android app-signing keystore file.
- `secrets.ANDROID_APP_SIGNING_KEY_PASSWORD`: Password for the app-signing key
  alias in the Android keystore.

**Required in `signing` environment when `build_android_aab` is `true`**

- `secrets.ANDROID_UPLOAD_KEYSTORE_B64`: Base64-encoded Android upload keystore
  (PKCS12 or JKS) used by Gradle when building release App Bundles.
- `secrets.ANDROID_UPLOAD_KEYSTORE_PASSWORD`: Password protecting the Android
  upload keystore file.
- `secrets.ANDROID_UPLOAD_KEY_PASSWORD`: Password for the upload key alias in
  the Android keystore.

**Required in `signing` environment when `build_ios_ipa` is `true`**

- `secrets.IOS_SIGNING_CERT_B64`: Base64-encoded iOS distribution signing
  certificate (`.p12`) for App Store builds.
- `secrets.IOS_SIGNING_CERT_PASSWORD`: Password protecting the iOS signing
  certificate.
- `secrets.IOS_PROVISIONING_PROFILE_B64`: Base64-encoded iOS provisioning
  profile for App Store builds.

---

## upload-build-android.yml

Uploads an Android App Bundle (`.aab`) from a release to Google Play via
Workload Identity Federation (WIF).

**Inputs**

| Name           | Required | Default  |
| -------------- | -------- | -------- |
| release_tag    | true     | —        |
| track          | false    | internal |
| release_status | false    | draft    |

**Environment Configuration**

This workflow reads Google Play Console publishing configuration from the
`release` environment in the calling repository.

**Required in `release` environment**

- `vars.GCP_WIF_PROVIDER`: Full resource name of the Google Cloud Workload
  Identity Federation provider
  (`projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/POOL_ID/providers/PROVIDER_ID`).
- `vars.GCP_SERVICE_ACCOUNT`: Email address of the Google Cloud service account
  to impersonate via Workload Identity Federation.
- `vars.GOOGLE_PLAY_PACKAGE_NAME`: Android application ID in Google Play (for
  example `com.earthmanmuons.whatchord`).

---

## upload-build-ios.yml

Uploads an iOS IPA from a release to App Store Connect using API key
authentication.

**Inputs**

| Name        | Required | Default |
| ----------- | -------- | ------- |
| release_tag | true     | —       |

**Environment Configuration**

This workflow reads App Store Connect authentication settings from the `release`
environment in the calling repository.

**Required in `release` environment**

- `vars.ASC_ISSUER_ID`: App Store Connect issuer identifier.
- `vars.ASC_KEY_ID`: App Store Connect API key identifier.
- `secrets.ASC_AUTH_KEY_B64`: Base64-encoded App Store Connect private key
  (`AuthKey_*.p8`).
