# Slice S-00-05 — Edge (Go) builds and proves alive

> Status: Proposed
> Depends on: S-00-01
> Size: ≤ ½ day

## Outcome / why now
The Train Edge Gateway service builds and answers a health check under test, establishing the edge runtime as a first-class, buildable container target.
Serves: §6.4 (edge gateway in Go), D-19 (the same production images run on the rig — so the edge service is containerised from the start).

## Situation
The edge node is its own repository (`dream-edge`) with its own Go toolchain. Its module, a health handler, and a test must exist before any device-shadow or sync work.

## Scope
**In:**
- Go module at the root of the `dream-edge` repo; a trivial health / `hello` HTTP handler.
- The `dream-contracts` submodule (wired in S-00-01) is present and pinned; no codegen against it yet (S-02).
- `go test ./...` passes (handler returns healthy).
- Builds as a container image (Dockerfile), honouring the same-images guardrail.

**Out (deferred — and behind which seam):**
- Device-shadow store, local event log, sync endpoint → **S-02** / M1b / M2.
- BLE bindings, embedded SQLite store → later edge slices.

## Seams & contracts
- `dream-edge` repo Go module + `go.mod`
- The pinned `dream-contracts` submodule
- The `dream-edge` repo Dockerfile
- The Go health test

## Acceptance criteria
- Given the dream-edge repo, when `go build` and `go test ./...` run, then both pass and the health handler returns healthy.

## Definition of Done
- Green; alive proven by an automated test; the container image builds.

## Open decisions
- Pinned Go version.

## Converges into
The edge Go module, its Dockerfile, and the health test.
