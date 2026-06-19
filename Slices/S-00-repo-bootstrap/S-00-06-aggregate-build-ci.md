# Slice S-00-06 — dream-orchestration `make verify` / `make train` across repos + minimal CI

> Status: Proposed
> Depends on: S-00-02, S-00-03, S-00-04, S-00-05
> Size: ≤ ½ day

## Outcome / why now
One command in the dream-orchestration repo builds and tests all four stacks across their separate repositories, and CI runs it on every push — the single green heartbeat for the whole platform, preserved across the multi-repo split.
Serves: MVP §6 (M0 gate — `make`-driven, zero external systems), MVP Technical Choices (dream-orchestration repo owns `make train`, checking out sibling repos).

## Situation
The four stacks build individually in their own repos on their own toolchains. With no monorepo to give a single checkout, the dream-orchestration repo is the cross-repo entry point: it must check out each sibling at a pinned SHA, build and test them together, and prove them green on every push.

## Scope
**In:**
- dream-orchestration `Makefile` `verify` target: checks out each sibling repo at its **pinned SHA** (using the S-00-01 manifest/script), then invokes each stack's own build + test — the backend Gradle build (`./gradlew build`), the web npm/Vite build + Vitest, the Android Gradle build (`./gradlew assembleDebug` + test), and the edge `go build` + `go test ./...`.
- dream-orchestration `Makefile` `train` skeleton wired to the same checkout mechanism (full bring-up filled in S-03; here it resolves siblings and is a no-op/placeholder for the sim).
- Minimal `.gitlab-ci.yml` in **each stack repo** running that repo's build + test on push.
- Minimal `.gitlab-ci.yml` in the **dream-orchestration repo** running `make verify` across the pinned siblings on push. Build + test only.

**Out (deferred — and behind which seam):**
- CI hardening — no-network guarantee, lint, Trivy, Pact — and the Terraform skeleton → **S-01**.
- The actual `make train` bring-up of the sim + edge gateway → **S-03**.
- Switching cross-repo bring-up from build-from-source to pulling published images → deferred dream-orchestration decision.

## Seams & contracts
- `dream-orchestration/Makefile` (`verify` target; `train` skeleton) + sibling-checkout script + pinned-SHA manifest
- `.gitlab-ci.yml` in each stack repo
- `dream-orchestration/.gitlab-ci.yml`

## Acceptance criteria
- Given clean clones, when `make verify` runs in the dream-orchestration repo, then it checks out each sibling at its pinned SHA and all four stacks build and their smoke tests pass.
- Given a push to any stack repo, when its CI runs, then that repo's build + test pipeline is green on a runner with no project-specific external systems.
- Given a push to the dream-orchestration repo, when CI runs `make verify`, then the cross-repo pipeline is green on a runner with no project-specific external systems.

## Definition of Done
- Green on trunk in every repo; one dream-orchestration command covers all stacks across repos; zero external systems in CI.
- Sibling SHAs are pinned and recorded, so the cross-repo build is reproducible.

## Open decisions
- How `make verify` obtains siblings on a fresh CI runner (submodule of siblings vs scripted clone at pinned SHA) — recorded in the dream-orchestration repo.

## Converges into
The `dream-orchestration` `Makefile` (+ checkout script + pinned-SHA manifest), the per-repo `.gitlab-ci.yml` files, and the dream-orchestration `.gitlab-ci.yml`.
