# Slice S-00-03 — Operator Android app (Kotlin/Compose): own repo, hello screen, assembles and proves alive, own CI

> Status: Ready
> Depends on: —
> Size: ≤ ½ day

## Outcome / why now
The `dream-operator-android-app` repo exists on trunk with a hello-world Kotlin/Compose app that assembles a debug build, passes a smoke test, and is kept green by its own CI — giving the staff surface its first independently green build on the managed-device toolchain.
Serves: D-10 (Android-only operator app), MVP Technical Choices (Kotlin + Jetpack Compose; Gradle).

## Situation
The Android app is the next stack we stand up end to end, on its own. It is a Kotlin/Compose stack in its own repository (`dream-operator-android-app`), inherently decoupled from the backend so the Android Gradle Plugin's Gradle/JDK constraints stay contained to this app. This subslice creates the repo, the build, a hello screen, the smoke test, and the repo's own CI together so the surface is independently green before any conductor features. It does not touch contracts or other repos.

## Scope
**In:**
- **Create the `dream-operator-android-app` repo on trunk** with a root `.gitignore` and `README.md`.
- **Self-contained Gradle build at the repo root** — its own `settings.gradle.kts`, `build.gradle.kts`, version catalog (`gradle/libs.versions.toml`), committed `gradlew` wrapper; AGP / Gradle / JDK / compile-target SDK pinned here.
- **Kotlin + Jetpack Compose hello-world app;** a hello screen.
- **`assembleDebug` succeeds.**
- **One JUnit smoke test passes.**
- **The repo's own minimal GitHub Actions workflow (`.github/workflows/ci.yml`):** build + test on every push (`./gradlew assembleDebug` + test). Build + test only; no gates.

**Out (deferred — and behind which seam):**
- `dream-contracts` repo + git-submodule wiring → **S-02** (this repo is standalone; no submodule in S-00).
- CI hardening — lint, Trivy → **S-01**.
- Offline ticket validation, QR (CameraX + ML Kit), MDM integration → later operator-app slices.

## Seams & contracts
- `dream-operator-android-app` repo root `.gitignore`, `README.md`
- Gradle build — `settings.gradle.kts`, `build.gradle.kts`, `gradle/libs.versions.toml`, `gradlew`
- The Compose hello screen
- The JUnit smoke test
- `dream-operator-android-app/.github/workflows/ci.yml` (build + test)

## Acceptance criteria
- Given a clean clone, when `./gradlew assembleDebug` runs at the repo root, then it succeeds; and when its JUnit smoke test runs, then it passes.
- Given a push to `dream-operator-android-app`, when its CI runs on a runner with no project-specific external systems, then the build + test pipeline is green.

## Definition of Done
- On trunk; green via the repo's own CI; alive proven by an automated test; zero external systems.
- The Android build is independent — AGP's Gradle/JDK constraints are contained to this app, not imposed on the backend; no submodule.
- Toolchain pinned (AGP / Kotlin / Gradle / JDK / compile-target SDK) so the build is reproducible across machines and CI.

## Open decisions
- AGP / Kotlin / Gradle / JDK / compile-target SDK versions — recorded in this app's version catalog.

## Converges into
The `dream-operator-android-app` repo, its Gradle build + wrapper, the hello-world Compose app, the JUnit smoke test, and the repo's GitHub Actions workflow (`.github/workflows/ci.yml`).
