# Slice S-00-05 — `dream-orchestration` repo: wrap the four stacks into one `make verify` / `make train` + cross-repo CI

> Status: Proposed
> Depends on: S-00-01, S-00-02, S-00-03, S-00-04
> Size: ≤ ½ day

## Outcome / why now
With all four stacks already standing on their own and individually green, the `dream-orchestration` repo is created last to wrap them up: one command checks out each sibling at a pinned SHA, builds and tests all four across their separate repositories, and CI runs it on every push — the single green heartbeat for the whole platform, layered on top of the per-repo heartbeats rather than replacing them.
Serves: MVP §6 (M0 gate — `make`-driven, zero external systems), MVP Technical Choices (dream-orchestration repo owns `make train`, checking out sibling repos).

## Situation
The four stacks build and test individually in their own repos on their own toolchains, each kept green by its own CI (S-00-01…04). With no monorepo to give a single checkout, the `dream-orchestration` repo is the cross-repo entry point and the final piece of S-00: it must check out each of the four now-existing siblings at a pinned SHA, build and test them together, and prove them green on every push. It is created only after the four stacks exist, so its pinned-SHA manifest references real, green trunk commits.

## Scope
**In:**
- **Create the `dream-orchestration` repo on trunk** with a root `.gitignore` and `README.md`.
- **`Makefile` `verify` target:** checks out each of the four sibling repos at its **pinned SHA** (via a sibling-checkout script + pinned-SHA manifest in this repo), then invokes each stack's own build + test — the backend Gradle build (`./gradlew build`), the web npm/Vite build + Vitest, the Android Gradle build (`./gradlew assembleDebug` + test), and the edge `go build` + `go test ./...`.
- **`Makefile` `train` skeleton** wired to the same checkout mechanism (full bring-up filled in S-03; here it resolves siblings and is a no-op/placeholder for the sim).
- **Sibling-checkout script + pinned-SHA manifest** that knows the four repos and their pinned SHAs.
- **The repo's own `.gitlab-ci.yml`:** runs `make verify` across the pinned siblings on push. Build + test only; no gates.

**Out (deferred — and behind which seam):**
- `dream-contracts` repo + git-submodule wiring → **S-02**.
- `dream-infra` repo + Terraform skeleton → **S-01**.
- CI hardening — no-network guarantee, lint, Trivy, Pact → **S-01**.
- The actual `make train` bring-up of the sim + edge gateway → **S-03**.
- The conformance harness + rung-1 sim (this repo is their future home) → **S-02 / S-03**.
- Switching cross-repo bring-up from build-from-source to pulling published images → deferred dream-orchestration decision.

## Seams & contracts
- `dream-orchestration` repo root `.gitignore`, `README.md`
- `dream-orchestration/Makefile` (`verify` target; `train` skeleton)
- The sibling-checkout script + pinned-SHA manifest
- `dream-orchestration/.gitlab-ci.yml` (cross-repo `make verify`)

## Acceptance criteria
- Given clean clones, when `make verify` runs in the dream-orchestration repo, then it checks out each of the four siblings at its pinned SHA and all four stacks build and their smoke tests pass.
- Given a push to the dream-orchestration repo, when CI runs `make verify`, then the cross-repo pipeline is green on a runner with no project-specific external systems.
- Given the `train` target, when it runs, then it resolves the four siblings at their pinned SHAs (full sim bring-up not yet required).

## Definition of Done
- On trunk; green via its own CI; one dream-orchestration command covers all four stacks across repos; zero external systems in CI.
- Sibling SHAs are pinned and recorded, so the cross-repo build is reproducible.
- The per-repo heartbeats remain authoritative for each stack; this repo aggregates them, it does not replace them.

## Open decisions
- How `make verify` obtains siblings on a fresh CI runner (scripted clone at pinned SHA vs submodule-of-siblings) — recorded in the dream-orchestration repo.
- Where the pinned-SHA manifest lives (`Makefile` vars vs a separate file) — recorded in the dream-orchestration repo.

## Converges into
The `dream-orchestration` repo, its `Makefile` (`verify` target + `train` skeleton), the sibling-checkout script + pinned-SHA manifest, and its `.gitlab-ci.yml`.
