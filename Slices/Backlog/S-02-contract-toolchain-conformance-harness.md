# Slice S-02 — Establish the contract-first toolchain and the conformance harness skeleton

> Status: Proposed
> Depends on: S-01
> Size: 1–2 days

## Outcome / why now
Contracts become the generated source of truth and every train surrogate runs through one harness, so the five rungs cannot quietly drift — the failure mode §8.2 names as the killer of setups like this.
Serves: D-18 (contract-first), D-19 (no divergence), §8.2 (conformance suite is the keystone).

## Situation
When we generate clients, fakes, and conformance tests *from* machine-readable contracts, the contracts (now in the `dream-contracts` repo, consumed via the pinned submodule) and the harness (in the `dream-orchestration` repo) must precede the backend code they describe — otherwise the suite becomes a retrofit and the contracts become documentation written after the fact.

## Scope
**In:**
- In the `dream-contracts` repo: OpenAPI for the seven walking-skeleton operations only — `search`, `hold`, `pay`, `issue-ticket`, `validate`, `advance-state`, `query-state`; one AsyncAPI message for the edge↔backend sync the skeleton exercises; one device-shadow JSON Schema (the berth/facility shadow the skeleton touches).
- Each consumer repo bumps its pinned `dream-contracts` submodule SHA to adopt these specs.
- `openapi-generator` producing the typed server interface + clients in each consumer (against the pinned submodule); Spectral lint in the `dream-contracts` repo CI.
- Black-box conformance harness shell (in the `dream-orchestration` repo) with one trivial assertion, invoked through a **rung-agnostic** entry point (same call for in-memory fake, twin, rig).

**Out (deferred — and behind which seam):**
- Real assertions for the sync seam — behind the AsyncAPI contract until the M1b conflict-model decision.
- The full contract surface (only the skeleton path now).
- Pact / Schemathesis depth — Pact gate skeleton lands in S-01; depth added as real provider/consumer boundaries appear.

## Seams & contracts
- `dream-contracts` repo: `openapi/booking.yaml`, `asyncapi/sync.yaml`, `shadow/*.schema.json`
- Pinned `dream-contracts` submodule SHA in each consumer repo
- `dream-orchestration` repo: `conformance/harness/`

## Acceptance criteria
- Given clean clones with no network, when CI runs, then Spectral lints the specs in the `dream-contracts` repo, the generator emits the typed client in each consumer against the pinned submodule, and the dream-orchestration harness runs green against the rung-1 sim.
- Given a malformed spec, when CI runs, then the lint gate fails the build.
- Given a backend that violates the published contract, when the harness runs against it, then the harness fails.
- Given the harness, when pointed at any rung, then it is invoked by the identical entry point (no per-rung test code).

## Definition of Done
- Zero external systems in CI.
- Harness entry point is rung-agnostic from commit one.
- Specs are the generation source — no hand-written client checked in.

## Open decisions
- Provisional-sync model: the sync message's conflict behaviour is a placeholder until M1b. ADR to be opened (offline conflict resolution — CRDT vs event-log, §7 HIGH RISK).

## Converges into
The contract files, the generated clients, the conformance harness.
