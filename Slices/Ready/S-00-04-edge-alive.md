# Slice S-00-04 — Edge (Go): own repo, hello handler, builds and proves alive, container image, own CI

> Status: Proposed
> Depends on: —
> Size: ≤ ½ day

## Outcome / why now
The `dream-edge` repo exists on trunk with a hello-world Go service that builds, answers a health check under test, builds as a container image, and is kept green by its own CI — establishing the edge runtime as a first-class, buildable container target.
Serves: §6.4 (edge gateway in Go), D-19 (the same production images run on the rig — so the edge service is containerised from the start).

## Situation
The edge node is the last of the four stacks we stand up end to end, on its own. It is its own repository (`dream-edge`) with its own Go toolchain. This subslice creates the repo, the module, a health handler, the test, the Dockerfile, and the repo's own CI together so the edge runtime is independently green before any device-shadow or sync work. It does not touch contracts or other repos.

## Scope
**In:**
- **Create the `dream-edge` repo on trunk** with a root `.gitignore` and `README.md`.
- **Go module at the repo root**; pinned Go version; a trivial health / `hello` HTTP handler.
- **`go test ./...` passes** (handler returns healthy).
- **Builds as a container image** (Dockerfile), honouring the same-images guardrail (D-19).
- **The repo's own minimal GitHub Actions workflow (`.github/workflows/ci.yml`):** build + test on every push (`go build` + `go test ./...`). Build + test only; no gates.

**Out (deferred — and behind which seam):**
- `dream-contracts` repo + git-submodule wiring → **S-02** (this repo is standalone; no submodule in S-00).
- CI hardening — Trivy image scan, lint → **S-01**.
- Device-shadow store, local event log, sync endpoint → **S-02** / M1b / M2.
- BLE bindings, embedded SQLite store → later edge slices.

## Seams & contracts
- `dream-edge` repo root `.gitignore`, `README.md`
- Go module + `go.mod`
- The health / `hello` handler
- The Dockerfile
- The Go health test
- `dream-edge/.github/workflows/ci.yml` (build + test)

## Acceptance criteria
- Given a clean clone of `dream-edge`, when `go build` and `go test ./...` run, then both pass and the health handler returns healthy.
- Given the Dockerfile, when the image builds, then it builds successfully.
- Given a push to `dream-edge`, when its CI runs on a runner with no project-specific external systems, then the build + test pipeline is green.

## Definition of Done
- On trunk; green via the repo's own CI; alive proven by an automated test; the container image builds; zero external systems.
- The edge build is independent — its own repo and toolchain, no submodule.
- Go version pinned so the build is reproducible across machines and CI.

## Open decisions
- Pinned Go version — recorded in the repo.

## Converges into
The `dream-edge` repo, its Go module, the health/`hello` handler, the Dockerfile, the health test, and the repo's GitHub Actions workflow (`.github/workflows/ci.yml`).
