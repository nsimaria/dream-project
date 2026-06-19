# Slice S-00-04 — Operator Android app (Kotlin/Compose) assembles and proves alive

> Status: Proposed
> Depends on: S-00-01
> Size: ≤ ½ day

## Outcome / why now
The operator app shell assembles and passes a smoke test in its own repository, giving the staff surface its first green build on the managed-device toolchain.
Serves: D-10 (Android-only operator app), MVP Technical Choices (Kotlin + Jetpack Compose).

## Situation
The Android app is a Kotlin/Compose stack in its own repository (`dream-operator-android-app`), inherently decoupled from the backend so the Android Gradle Plugin's Gradle/JDK constraints stay contained to this app. It must assemble a debug build and pass a unit test before any conductor features.

## Scope
**In:**
- Self-contained Gradle build at the root of the `dream-operator-android-app` repo — its own `settings.gradle.kts`, `build.gradle.kts`, version catalog, committed wrapper; AGP / Gradle / JDK pinned here.
- The `dream-contracts` submodule (wired in S-00-01) is present and pinned; no codegen against it yet (S-02).
- Kotlin + Jetpack Compose app; a hello screen.
- `assembleDebug` succeeds.
- One JUnit smoke test passes.

**Out (deferred — and behind which seam):**
- Offline ticket validation, QR (CameraX + ML Kit), MDM integration → later operator-app slices.

## Seams & contracts
- `dream-operator-android-app` repo Gradle build — `settings.gradle.kts`, `build.gradle.kts`, `gradle/libs.versions.toml`, `gradlew`
- The pinned `dream-contracts` submodule
- The JUnit smoke test

## Acceptance criteria
- Given the Android repo, when `./gradlew assembleDebug` runs at its root, then it succeeds; and when its JUnit smoke test runs, then it passes.

## Definition of Done
- Green; alive proven by an automated test.
- The Android build is independent — AGP's Gradle/JDK constraints are contained to this app, not imposed on the backend.

## Open decisions
- AGP / Kotlin / Gradle / JDK / compile-target SDK versions — recorded in this app's version catalog.

## Converges into
The Android Gradle build + wrapper and its smoke test.
