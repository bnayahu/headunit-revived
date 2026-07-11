# Continuous Integration

## `android-ci.yml` — Android CI

Runs on every pull request and on pushes to `main`. Three parallel jobs:

| Check                        | What it does                                   | Blocks the PR? |
|------------------------------|------------------------------------------------|:--------------:|
| **Build (github debug)**     | `./gradlew :app:assembleGithubDebug` — compiles the app (including native code and the `:contract` module) and uploads the debug APK as an artifact. | Yes |
| **Unit tests (github debug)**| `./gradlew :app:testGithubDebugUnitTest` — runs the JVM unit tests and uploads the HTML/XML report. | Yes |
| **Lint (advisory)**          | `./gradlew :app:lintGithubDebug` — runs Android Lint and uploads the report. | No (advisory) |

The build is **debug, unsigned, and uses no repository secrets**, so it runs
unchanged on pull requests opened from forks.

### Branch protection

To make CI enforce quality on `main`, add a branch-protection rule requiring
these status checks:

- `Build (github debug)`
- `Unit tests (github debug)`

Leave **`Lint (advisory)`** unrequired — it reports findings (and uploads a
report artifact) but is intentionally non-blocking, matching the project's
`lint { abortOnError = false }` in `app/build.gradle.kts`.

### Notes

- **`github` flavor only.** The `playstore` flavor is not built because it
  cannot currently compile: `VpnControl.kt` lives only in the `github` source
  set (`app/src/github/...`) but is imported by shared `main` code. Once that
  split is resolved, add a `playstore` build to the `build` job.
- **`main` may show red** on `Build`/`Unit tests` until the known
  `SettingsFragment.kt:767` smart-cast compile error (issue #659) is fixed.
  This is expected, not a CI misconfiguration.
- **Toolchain:** Temurin JDK 21 (Gradle 8.13 rejects Java 24+), plus the pinned
  NDK `27.0.12077973` and CMake `3.22.1` installed via `sdkmanager`.
- **Action pinning:** third-party actions are pinned to major-version tags.
  Maintainers who want stricter supply-chain hygiene can repin them to full
  commit SHAs.
