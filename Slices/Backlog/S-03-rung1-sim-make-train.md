# Slice S-03 — Build the rung-1 in-memory train sim behind `make train`

> Status: Proposed
> Depends on: S-02
> Size: 1–2 days

## Outcome / why now
A developer spins up a full simulated train with one command and never needs hardware, so every feature is buildable and testable offline-first from day one — removing the three-year hardware gap as a blocker.
Serves: D-18 (developer experience as a first-class product), MVP §6 (M0 exit: `make train` + CI green with zero external systems).

_Note: this is a legitimate developer-as-user slice — the developer is the genuine consumer of the deliverable, so the value is real and non-circular._

## Situation
When no real trains exist for most of the build, the in-memory simulator is the always-on rung-1 backend behind the edge adapter contract — the fast CI/development sandbox every other slice runs against.

## Scope
**In:**
- In-memory implementation of the device-shadow adapter contract (S-02), in the `dream-orchestration` repo alongside the conformance harness — deterministic, runs in seconds.
- `make train` (dream-orchestration repo): checks out the `dream-edge` repo at its pinned SHA, brings up the sim + the Go edge gateway locally, no external systems.
- Wired into the conformance harness as rung 1.
- Seeded/deterministic behaviour for repeatable CI.

**Out (deferred — and behind which seam):**
- Connectivity-dropout and fault-injection depth — rung-2 software twin (separate slice, M1b).
- BLE flakiness, physical movement, multi-unit concurrency — rung-3 Lab.
- Real conflict resolution / reconciliation — M1b decision, behind the AsyncAPI contract.

## Seams & contracts
- Implements the device-shadow schemas from the `dream-contracts` repo (via the pinned submodule) and the edge adapter contract.
- `dream-orchestration` repo: `sim/`
- `dream-orchestration/Makefile` target `train`

## Acceptance criteria
- Given clean clones, when I run `make train` in the dream-orchestration repo, then it checks out the dream-edge repo at its pinned SHA and a simulated train + edge gateway come up locally with no external systems.
- Given the conformance harness pointed at rung 1, when it runs, then it passes.
- Given two `make train` runs with the same seed, when compared, then behaviour is identical (deterministic).

## Definition of Done
- One command, no external systems, deterministic.
- Runs in CI; rung 1 passes the harness from S-02.
- Same adapter contract as every higher rung — no sim-only shortcuts.

## Open decisions
- None.

## Converges into
The simulator code, the `make train` target, the rung-1 adapter implementation.
