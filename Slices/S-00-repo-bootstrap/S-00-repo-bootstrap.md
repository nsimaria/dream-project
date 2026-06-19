# Slice S-00 — Bootstrap the polyglot repos: every stack boots and proves itself alive

> Status: Proposed
> Depends on: —
> Size: 1–2 days

## Outcome / why now
Every engineer who follows gets a set of repositories — one per surface/component — that each build and test their own stack, share contracts through a pinned submodule, and come up together as one simulated train via the dream-orchestration repo, each stack proving it is alive with a test rather than by hand.
Serves: MVP §6 (M0 gate — `make`-driven, zero external systems), MVP Technical Choices (the four-language map: TS / Kotlin / Java / Go; multi-repo from day 1), D-19 (edge runs the same images everywhere — established here as a buildable Go service).

## Situation
We are starting multi-repo: one repository per product surface/component, plus a shared `dream-contracts` repo and a `dream-orchestration` repo (MVP Technical Choices — Repository approach). Before any boundary rule, schema, or contract can be enforced, each repo's runtime has to compile, boot, and answer "am I alive?" under its own reproducible toolchain — and the cross-repo bring-up (`make train`) and contract-sharing mechanism (submodule) have to exist so the rungs and the consumers cannot drift.

## Scope
**In:**
- **Seven repositories on trunk:** `dream-backend`, `dream-passenger-web-app`, `dream-operator-android-app`, `dream-edge`, `dream-contracts`, `dream-infra`, `dream-orchestration`. Each with its own `.gitignore` and `README`. (`Documents` / `Slices` planning content remains where it is — not a code repo concern here.)
- **Self-contained per-stack builds:** `dream-backend` and `dream-operator-android-app` each own an independent Gradle build + wrapper (no shared umbrella build), `dream-passenger-web-app` uses npm/Vite, `dream-edge` uses the Go toolchain; each pins its own tool versions for reproducibility.
- **`dream-contracts` shared by submodule:** the `dream-contracts` repo holds a placeholder layout (`openapi/`, `asyncapi/`, `shadow/`); each consuming repo embeds it as a **git submodule pinned at an explicit SHA**. (Codegen against it lands in S-02; here it is wired and pinned only.)
- **Backend (Java/Spring Boot):** a single bootable app (not yet decomposed into domain modules) exposing a liveness endpoint (Actuator `/health` or `/hello`); Spring Boot Test smoke test asserts context loads and the endpoint returns 200.
- **Web (React/TS, Vite PWA shell):** builds; renders a hello page; Vitest + RTL smoke test passes.
- **Android (Kotlin/Compose):** assembles `:debug`; one JUnit smoke test passes.
- **Edge (Go):** builds; trivial health/`hello` handler; `go test` passes.
- **`dream-orchestration` repo:** a `Makefile` with `verify` (checks out each sibling repo at a pinned SHA, builds and tests all four stacks) and the skeleton of `train`; cross-repo scripts; the home for the conformance harness and rung-1 sim (filled in S-02/S-03).
- **Minimal CI (GitLab CI):** each stack repo runs build + test on every push; the dream-orchestration repo runs `make verify` across the pinned siblings. Build + test only; no gates yet.

**Out (deferred — and behind which seam):**
- Backend module decomposition (`trips`, `inventory`, `orders/ticketing`, `payments`), schema-per-module, and boundary enforcement (Spring Modulith + ArchUnit) → **S-01**.
- CI hardening — no-network guarantee, lint, Trivy, Pact — and the Terraform skeleton → **S-01**.
- Postgres, the outbox table, Flyway baseline, Testcontainers → **S-01**.
- Machine-readable contract *content* + codegen + conformance harness assertions → **S-02** (the `dream-contracts` repo and its submodule wiring exist here; the specs and generation do not).
- Any real domain logic → M1a slices.

## Seams & contracts
- Seven repo roots, each with `.gitignore` and `README`
- `dream-backend` and `dream-operator-android-app`: each its own `settings.gradle.kts`, `build.gradle.kts`, version catalog, and `gradlew` wrapper
- `.gitmodules` in each consuming repo pinning the `dream-contracts` submodule at a SHA
- `dream-orchestration/Makefile` (`verify` aggregate target; `train` skeleton) and its sibling-checkout scripts
- `.gitlab-ci.yml` in each repo (build + test) and in `dream-orchestration` (cross-repo `make verify`)
- Per-stack smoke tests in each repo's test path

## Acceptance criteria
- Given clean clones of all repos, when the dream-orchestration repo's `make verify` runs, then it checks out each sibling at its pinned SHA, and all four stacks build and their smoke tests pass.
- Given the dream-backend repo, when its Spring Boot Test runs, then the context loads and the liveness endpoint returns 200.
- Given the web repo, when the Vitest smoke test runs, then it passes after a successful build; given the Android repo, when its JUnit smoke test runs, then it passes after `assembleDebug`; given the dream-edge repo, when `go test ./...` runs, then it passes after a successful build.
- Given a consuming repo, when it is cloned with `--recurse-submodules`, then the `dream-contracts` submodule resolves to its pinned SHA.
- Given a CI runner with no project-specific external systems, when each repo's pipeline (and the dream-orchestration `make verify`) runs, then it is green.

## Definition of Done
- Green on each repo's trunk; one command (dream-orchestration `make verify`) builds and tests every stack across repos.
- Each stack proves "alive" with an automated test, not by hand.
- `dream-contracts` is wired as a pinned submodule in every consumer; bringing a repo up resolves it deterministically.
- Toolchain pinned (Gradle wrapper + version catalog, Node/JDK/Go versions) per repo so builds are reproducible across machines and CI.
- Zero external systems in CI.

## Open decisions
- Pinned runtime versions (JDK, Node, Go, Android SDK/compile target) — record in each stack repo's own toolchain / version-catalog files; thin ADR only if contested.
- Repo host / group structure (one GitLab group, seven projects) — GitLab assumed (MVP Technical Choices: GitLab CI).
- Cloud provider/region is **not** decided here (S-01 owns the infra skeleton); CI uses a generic runner.

## Converges into
The seven repositories and their layouts, the per-stack builds (the backend and Android Gradle builds + wrappers, the web npm/Vite project, the edge Go module), the pinned `dream-contracts` submodule wiring, the `dream-orchestration` `Makefile` + cross-repo scripts, the per-repo and dream-orchestration `.gitlab-ci.yml`, and the four per-stack smoke tests.

## Breakdown
Delivered as subslices, one per piece:

| ID | Subslice | Depends on |
|---|---|---|
| S-00-01 | Repo set + `dream-contracts` repo + dream-orchestration skeleton + submodule wiring | — |
| S-00-02 | Backend (Java/Spring Boot) boots + alive test (own repo + Gradle build) | S-00-01 |
| S-00-03 | Passenger web app (React/TS) builds + alive test (own repo) | S-00-01 |
| S-00-04 | Operator Android app (Kotlin/Compose) assembles + alive test (own repo + Gradle build) | S-00-01 |
| S-00-05 | Edge (Go) builds + alive test (own repo) | S-00-01 |
| S-00-06 | dream-orchestration `make verify` / `make train` across repos + minimal CI | S-00-02…05 |
