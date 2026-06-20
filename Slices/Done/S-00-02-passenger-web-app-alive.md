# Slice S-00-02 — Passenger web app (React/TS): own repo, hello page, builds and proves alive, own CI

> Status: Proposed
> Depends on: —
> Size: ≤ ½ day

## Outcome / why now
The `dream-passenger-web-app` repo exists on trunk with a hello-world React/TS Vite shell that builds, renders, and is guarded by a smoke test and its own CI — giving the passenger surface its first independently green build before any booking UI.
Serves: MVP Technical Choices (React + TS PWA), D-08 (the accessible-primitive ecosystem is established as buildable here; the axe-core gate lands with S-01 CI hardening).

## Situation
The web surface is the next stack we stand up end to end, on its own and independent of the others. It needs its own repository (`dream-passenger-web-app`) with a Vite/TS project, a hello page, a passing test, and its own CI before any booking UI — created together so the surface is independently green the moment it lands. It does not touch contracts or other repos.

## Scope
**In:**
- **Create the `dream-passenger-web-app` repo on trunk** with a root `.gitignore` and `README.md`.
- **React + TS via Vite at the repo root**; PWA shell scaffolding (service-worker wiring minimal/deferred); pinned Node version.
- **A hello page renders.**
- **Vitest + React Testing Library smoke test passes.**
- **The repo's own minimal GitHub Actions workflow (`.github/workflows/ci.yml`):** build + test on every push (npm/Vite build + Vitest). Build + test only; no gates.

**Out (deferred — and behind which seam):**
- `dream-contracts` repo + git-submodule wiring → **S-02** (this repo is standalone; no submodule in S-00).
- Booking / seat-berth selection UI → M1a.
- axe-core accessibility gate, lint hardening → **S-01** CI hardening.
- Reaching the Train Edge Gateway over Lab Wi-Fi → later edge slices.

## Seams & contracts
- `dream-passenger-web-app` repo root `.gitignore`, `README.md`
- The Vite/TS project + `package.json`
- The hello page component
- The Vitest smoke test
- `dream-passenger-web-app/.github/workflows/ci.yml` (build + test)

## Acceptance criteria
- Given a clean checkout, when the web build runs, then it produces a bundle and the Vitest smoke test passes.
- Given a push to `dream-passenger-web-app`, when its CI runs on a runner with no project-specific external systems, then the build + test pipeline is green.

## Definition of Done
- On trunk; green via the repo's own CI; alive proven by an automated test; zero external systems.
- The web build is independent — its own repo and toolchain, no submodule.
- Node version pinned so the build is reproducible across machines and CI.

## Open decisions
- Pinned Node version — recorded in the project / toolchain files.

## Converges into
The `dream-passenger-web-app` repo, its Vite/TS project, the hello page, the Vitest smoke test, and the repo's GitHub Actions workflow (`.github/workflows/ci.yml`).
