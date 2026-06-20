# Slice S-00 — Bootstrap the polyglot repos one at a time: each stack stands up its own repo, boots, and proves itself alive

> Status: Proposed
> Depends on: —
> Size: 1–2 days

## Outcome / why now
Every engineer who follows gets a set of repositories — one per surface/component — created one at a time, each standing up its own repo with a hello-world app and its own CI before the next begins, so that each stack proves it is alive with a test under its own reproducible toolchain, and only at the very end are they wrapped together into the `dream-orchestration` repo that brings them up as one simulated train.
Serves: MVP §6 (M0 gate — `make`-driven, zero external systems), MVP Technical Choices (the four-language map: TS / Kotlin / Java / Go; multi-repo from day 1), D-19 (edge runs the same images everywhere — established here as a buildable Go service).

## Situation
We are starting multi-repo: one repository per product surface/component, plus (later) a shared `dream-contracts` repo and the `dream-orchestration` repo (MVP Technical Choices — Repository approach). Rather than creating all repos up front in one big-bang slice, we stand each stack up end to end on its own — repo, hello-world runtime, and that repo's own CI — one after another, so each is independently green before the next starts. Before any boundary rule, schema, or contract can be enforced, each repo's runtime has to compile, boot, and answer "am I alive?" under its own reproducible toolchain. Only once all four stacks exist and are individually green do we add the cross-repo bring-up (`make verify` / `make train`) in `dream-orchestration` so the rungs and the consumers cannot drift.

## Scope
**In:**
- **Five repositories on trunk, created one at a time:** `dream-backend`, `dream-passenger-web-app`, `dream-operator-android-app`, `dream-edge` (the four stacks), then `dream-orchestration` (the cross-repo wrap-up, created last). Each with its own `.gitignore` and `README`. (`Documents` / `Slices` planning content remains where it is — not a code repo concern here.)
- **Self-contained per-stack builds, each with its own CI:** every stack repo owns its toolchain, its hello-world runtime, its alive test, and its own minimal GitHub Actions workflow (`.github/workflows/ci.yml`, build + test on push) — added in the same subslice that creates the repo, so each repo is independently green the moment it lands. `dream-backend` and `dream-operator-android-app` each own an independent Gradle build + wrapper (no shared umbrella build), `dream-passenger-web-app` uses npm/Vite, `dream-edge` uses the Go toolchain; each pins its own tool versions for reproducibility.
- **Backend (Java/Spring Boot):** a single bootable app (not yet decomposed into domain modules) exposing a liveness endpoint (Actuator `/health` or `/hello`); Spring Boot Test smoke test asserts context loads and the endpoint returns 200.
- **Web (React/TS, Vite PWA shell):** builds; renders a hello page; Vitest + RTL smoke test passes.
- **Android (Kotlin/Compose):** assembles `:debug`; one JUnit smoke test passes.
- **Edge (Go):** builds; trivial health/`hello` handler; `go test` passes; container image builds.
- **`dream-orchestration` repo (created last):** a `Makefile` with `verify` (checks out each of the four now-existing sibling repos at a pinned SHA, builds and tests all four stacks) and the skeleton of `train`; cross-repo scripts; a pinned-SHA manifest; the home for the conformance harness and rung-1 sim (filled in S-02/S-03). Its own GitHub Actions workflow (`.github/workflows/ci.yml`) runs `make verify` across the pinned siblings.

**Out (deferred — and behind which seam):**
- **`dream-contracts` repo + git-submodule wiring in each consumer** → **S-02** (created when contract content exists; each app repo is fully standalone in S-00, no submodule).
- **`dream-infra` repo + Terraform skeleton** → **S-01**.
- Backend module decomposition (`trips`, `inventory`, `orders/ticketing`, `payments`), schema-per-module, and boundary enforcement (Spring Modulith + ArchUnit) → **S-01**.
- CI hardening — no-network guarantee, lint, Trivy, Pact → **S-01** (S-00 CI is build + test only, no gates).
- Postgres, the outbox table, Flyway baseline, Testcontainers → **S-01**.
- Machine-readable contract *content* + codegen + conformance harness assertions → **S-02**.
- The actual `make train` bring-up of the sim + edge gateway → **S-03**.
- Any real domain logic → M1a slices.

## Seams & contracts
- Five repo roots, each with `.gitignore` and `README`
- `dream-backend` and `dream-operator-android-app`: each its own `settings.gradle.kts`, `build.gradle.kts`, version catalog, and `gradlew` wrapper
- `dream-passenger-web-app`: `package.json` + Vite/TS project; `dream-edge`: `go.mod` + Dockerfile
- GitHub Actions workflow (`.github/workflows/ci.yml`) in **each** stack repo (build + test), added with the repo
- `dream-orchestration/Makefile` (`verify` aggregate target; `train` skeleton), its sibling-checkout script + pinned-SHA manifest, and its GitHub Actions workflow (`.github/workflows/ci.yml`, cross-repo `make verify`)
- Per-stack smoke tests in each repo's test path

## Acceptance criteria
- Given each stack repo on its own, when its own CI runs on push, then that repo builds and its smoke test passes with zero project-specific external systems — independently, before any later repo exists.
- Given the dream-backend repo, when its Spring Boot Test runs, then the context loads and the liveness endpoint returns 200.
- Given the web repo, when the Vitest smoke test runs, then it passes after a successful build; given the Android repo, when its JUnit smoke test runs, then it passes after `assembleDebug`; given the dream-edge repo, when `go test ./...` runs, then it passes after a successful build and its container image builds.
- Given clean clones of all repos, when the dream-orchestration repo's `make verify` runs, then it checks out each of the four siblings at its pinned SHA, and all four stacks build and their smoke tests pass.
- Given a CI runner with no project-specific external systems, when each repo's pipeline (and the dream-orchestration `make verify`) runs, then it is green.

## Definition of Done
- Green on each repo's trunk; each stack repo is independently green via its own CI the moment it lands.
- Each stack proves "alive" with an automated test, not by hand.
- One command (dream-orchestration `make verify`) builds and tests every stack across repos, added only after all four stacks exist.
- Toolchain pinned (Gradle wrapper + version catalog, Node/JDK/Go versions) per repo so builds are reproducible across machines and CI.
- Zero external systems in CI.

## Open decisions
- Pinned runtime versions (JDK, Node, Go, Android SDK/compile target) — record in each stack repo's own toolchain / version-catalog files; thin ADR only if contested.
- Repo host / org structure (one GitHub org, five repos in S-00) — GitHub assumed (MVP Technical Choices: GitHub Actions).
- Cloud provider/region is **not** decided here (S-01 owns the infra skeleton); CI uses a generic runner.
- How `make verify` obtains siblings on a fresh CI runner (scripted clone at pinned SHA vs submodule-of-siblings) — recorded in the dream-orchestration repo.

## Converges into
The five repositories and their layouts, the per-stack builds each with its own CI (the backend and Android Gradle builds + wrappers, the web npm/Vite project, the edge Go module + Dockerfile), the four per-stack smoke tests and per-repo GitHub Actions workflows (`.github/workflows/ci.yml`), and — created last — the `dream-orchestration` `Makefile` + cross-repo scripts + pinned-SHA manifest and its GitHub Actions workflow (`.github/workflows/ci.yml`).

## Breakdown
Delivered as subslices, one per repo, built one at a time and each independently green before the next:

| ID | Subslice | Depends on |
|---|---|---|
| S-00-01 | Backend (Java/Spring Boot) — own repo + Gradle build + hello-world app + alive test + own CI | — |
| S-00-02 | Passenger web app (React/TS) — own repo + Vite build + hello page + alive test + own CI | — |
| S-00-03 | Operator Android app (Kotlin/Compose) — own repo + Gradle build + hello screen + alive test + own CI | — |
| S-00-04 | Edge (Go) — own repo + Go module + hello handler + alive test + Dockerfile + own CI | — |
| S-00-05 | `dream-orchestration` repo — `make verify` / `make train` across the four repos + cross-repo CI | S-00-01…04 |
