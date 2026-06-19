# Slice S-00-03 — Passenger web app (React/TS) builds and proves alive

> Status: Proposed
> Depends on: S-00-01
> Size: ≤ ½ day

## Outcome / why now
The passenger web shell builds and renders, with a smoke test guarding that the first-party surface is alive.
Serves: MVP Technical Choices (React + TS PWA), D-08 (the accessible-primitive ecosystem is established as buildable here; the axe-core gate lands with S-01 CI hardening).

## Situation
The web surface needs its own repository (`dream-passenger-web-app`) with a Vite/TS project, a hello page, and a passing test before any booking UI.

## Scope
**In:**
- React + TS via Vite at the root of the `dream-passenger-web-app` repo; PWA shell scaffolding (service-worker wiring minimal/deferred).
- The `dream-contracts` submodule (wired in S-00-01) is present and pinned; no codegen against it yet (S-02).
- A hello page renders.
- Vitest + React Testing Library smoke test passes.

**Out (deferred — and behind which seam):**
- Booking / seat-berth selection UI → M1a.
- axe-core accessibility gate → **S-01** CI hardening.
- Reaching the Train Edge Gateway over Lab Wi-Fi → later edge slices.

## Seams & contracts
- `dream-passenger-web-app` repo project + `package.json`
- The pinned `dream-contracts` submodule
- The Vitest smoke test

## Acceptance criteria
- Given a clean checkout, when the web build runs, then it produces a bundle and the Vitest smoke test passes.

## Definition of Done
- Green; alive proven by an automated test.

## Open decisions
- Pinned Node version (record in the project / toolchain files).

## Converges into
The web project and its smoke test.
