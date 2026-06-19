# Slice S-00-02 — Backend (Java/Spring Boot) boots and proves alive

> Status: Proposed
> Depends on: S-00-01
> Size: ≤ ½ day

## Outcome / why now
The backend app boots and answers a liveness check under test in its own repository, giving the service tier its first green heartbeat.
Serves: MVP Technical Choices (Java/Spring Boot; Gradle), D-19 (the service that later runs the same images everywhere starts here as a buildable app).

## Situation
The backend is a JVM/Spring stack in its own repository (`dream-backend`) with its own toolchain, inherently decoupled at the build level from the Android app so its JDK and Gradle versions are never held hostage to the Android Gradle Plugin. A single Spring Boot app (not yet decomposed) must build in its own repo and prove liveness so later module work has a running host.

## Scope
**In:**
- Self-contained Gradle build at the root of the `dream-backend` repo — its own `settings.gradle.kts`, `build.gradle.kts`, version catalog, committed wrapper; JDK 21+ pinned.
- Single Spring Boot app (not yet decomposed into domain modules).
- The `dream-contracts` submodule (wired in S-00-01) is present and pinned; no codegen against it yet (S-02).
- Liveness endpoint — Actuator `/health` or `/hello`.
- Spring Boot Test smoke test: context loads and the endpoint returns 200 (MockMvc).

**Out (deferred — and behind which seam):**
- Module decomposition (`trips`, `inventory`, `orders/ticketing`, `payments`) + boundary rules → **S-01**.
- Postgres / Flyway / outbox → **S-01**.
- Domain logic → M1a.

## Seams & contracts
- `dream-backend` repo Gradle build — `settings.gradle.kts`, `build.gradle.kts`, `gradle/libs.versions.toml`, `gradlew`
- The pinned `dream-contracts` submodule
- The smoke test under `src/test`

## Acceptance criteria
- Given the dream-backend repo, when its Spring Boot Test runs, then the context loads and the liveness endpoint returns 200.
- Given `./gradlew build` run at the dream-backend repo root, when it runs, then it compiles and the smoke test passes.

## Definition of Done
- Green; alive proven by an automated test; zero external systems.
- The backend build is independent — its own repo and toolchain, no Gradle coupling to the Android app.

## Open decisions
- Pinned JDK / Spring Boot / Gradle versions — recorded in the backend's version catalog.

## Converges into
The backend Gradle build + wrapper and its liveness smoke test.
