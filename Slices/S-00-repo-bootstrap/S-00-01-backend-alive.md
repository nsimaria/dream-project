# Slice S-00-01 — Backend (Java/Spring Boot): own repo, hello-world app, boots and proves alive, own CI

> Status: Proposed
> Depends on: —
> Size: ≤ ½ day

## Outcome / why now
The `dream-backend` repo exists on trunk with a hello-world Spring Boot app that boots, answers a liveness check under test, and is kept green by its own CI — giving the service tier its first independently green heartbeat and a known repo home for later module work.
Serves: MVP §6 (M0), MVP Technical Choices (Java/Spring Boot; Gradle; multi-repo from day 1), D-19 (the service that later runs the same images everywhere starts here as a buildable app).

## Situation
Nothing exists yet. The backend is the first stack we stand up end to end: its own repository (`dream-backend`) with its own toolchain, inherently decoupled at the build level from the Android app so its JDK and Gradle versions are never held hostage to the Android Gradle Plugin. This subslice creates the repo, the build, a single bootable Spring Boot app, the alive test, and the repo's own CI — all in one go — so the service tier is independently green before any other repo exists. It does not touch contracts or other repos.

## Scope
**In:**
- **Create the `dream-backend` repo on trunk** with a root `.gitignore` and `README.md`.
- **Self-contained Gradle build at the repo root** — its own `settings.gradle.kts`, `build.gradle.kts`, version catalog (`gradle/libs.versions.toml`), committed `gradlew` wrapper; JDK 21+ pinned.
- **Single hello-world Spring Boot app** (not yet decomposed into domain modules) exposing a liveness endpoint — Actuator `/health` or `/hello`.
- **Spring Boot Test smoke test:** context loads and the endpoint returns 200 (MockMvc).
- **The repo's own minimal `.gitlab-ci.yml`:** build + test on every push (`./gradlew build`). Build + test only; no gates.

**Out (deferred — and behind which seam):**
- `dream-contracts` repo + git-submodule wiring → **S-02** (this repo is standalone; no submodule in S-00).
- Module decomposition (`trips`, `inventory`, `orders/ticketing`, `payments`) + boundary rules (Spring Modulith + ArchUnit) → **S-01**.
- CI hardening — no-network guarantee, lint, Trivy, Pact → **S-01**.
- Postgres / Flyway / outbox / Testcontainers → **S-01**.
- Domain logic → M1a.

## Seams & contracts
- `dream-backend` repo root `.gitignore`, `README.md`
- Gradle build — `settings.gradle.kts`, `build.gradle.kts`, `gradle/libs.versions.toml`, `gradlew`
- The liveness endpoint handler
- The smoke test under `src/test`
- `dream-backend/.gitlab-ci.yml` (build + test)

## Acceptance criteria
- Given a clean clone of `dream-backend`, when `./gradlew build` runs at the repo root, then it compiles and the smoke test passes.
- Given the running app, when its Spring Boot Test runs, then the context loads and the liveness endpoint returns 200.
- Given a push to `dream-backend`, when its CI runs on a runner with no project-specific external systems, then the build + test pipeline is green.

## Definition of Done
- On trunk; green via the repo's own CI; alive proven by an automated test; zero external systems.
- The backend build is independent — its own repo and toolchain, no Gradle coupling to the Android app, no submodule.
- Toolchain pinned (Gradle wrapper + version catalog, JDK) so the build is reproducible across machines and CI.

## Open decisions
- Pinned JDK / Spring Boot / Gradle versions — recorded in the backend's version catalog.
- Liveness endpoint choice (Actuator `/health` vs a hand-rolled `/hello`) — recorded in the repo.

## Converges into
The `dream-backend` repo, its Gradle build + wrapper, the hello-world Spring Boot app + liveness endpoint, the liveness smoke test, and the repo's `.gitlab-ci.yml`.
