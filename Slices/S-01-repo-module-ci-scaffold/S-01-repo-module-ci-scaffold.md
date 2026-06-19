# Slice S-01 — Module boundaries, CI gates, and infra skeleton on the bootstrapped repos

> Status: Proposed
> Depends on: S-00
> Size: 1–2 days

## Outcome / why now
The running platform — and every later slice — gets repos where any change is gated against module boundaries and runs with no external systems, so the database-per-service ownership rule and the green-CI invariant are born enforced rather than retrofitted onto the working stacks S-00 stood up.
Serves: D-01 (module boundaries), D-03 (cloud-agnostic IaC), MVP §6 (M0 gate), MVP §4 (not-negotiable: schema boundaries).

## Situation
S-00 leaves the stacks each in their own repo, building, booting, and proving themselves alive, with per-repo and dream-orchestration pipelines already green. What those repos do **not** yet have is the structure they must respect: the `dream-backend` is still one undivided app, nothing stops a module reading another's schema, CI proves "it compiles," not "the boundaries hold and nothing reached the network," and no contract-drift gate (Pact) exists yet. Those properties are brutal to add back once feature code assumes their absence, so they go in now, directly on the green repos.

## Scope
**In:**
- Decompose the `dream-backend` repo's app into modules `trips`, `inventory`, `orders/ticketing`, `payments`, one Postgres schema each (no domain logic yet).
- Spring Modulith + ArchUnit rule: no module reads another's schema or package internals.
- Harden the S-00 pipelines (per repo + dream-orchestration): enforce no network access during build/test, add lint, add Trivy scan on container/IaC, add **Pact** consumer-driven contract checks against the pinned `dream-contracts` submodule.
- Terraform skeleton for one EU-region cluster in the `dream-infra` repo — `validate` only, no `apply`.
- Postgres with schema-per-module + the outbox table co-located per module (empty), Flyway baseline, Testcontainers, in the `dream-backend` repo.

**Out (deferred — and behind which seam):**
- Repo bring-up, per-stack hello-world + alive tests, the dream-orchestration `make verify` / `make train` (done in **S-00**).
- Real domain logic (later M1a slices).
- `terraform apply` / live cluster (later platform slice).
- ArgoCD/Flux GitOps (MVP §5 pushback #2 — arrives with the second deployable).
- Event broker (none in MVP — outbox + in-process relay; D-02).

## Seams & contracts
- `dream-backend` repo: `settings.gradle.kts`, `build.gradle.kts`
- Each repo's GitHub Actions workflows under `.github/workflows/` (+ `dream-orchestration`'s)
- `dream-infra` repo: `terraform/`
- ArchUnit/Modulith rules in the `dream-backend` repo's `src/test`
- `dream-backend` repo `**/db/migration` (Flyway)
- Pact contracts/verification per consumer repo, against the pinned `dream-contracts` submodule

## Acceptance criteria
- Given the S-00 pipelines, when they run with the network blocked, then build + unit tests still pass and the lint + Trivy gates pass.
- Given a commit that makes one backend module read another's schema or internal package, when CI runs, then the ArchUnit/Modulith boundary test fails the build.
- Given a consumer repo whose code has drifted from the pinned `dream-contracts`, when its CI runs, then the Pact contract check fails the build.
- Given the infra skeleton, when `terraform validate` runs, then it passes without any `apply`.
- Given a backend module, when it starts under Testcontainers Postgres, then its Flyway baseline (including the outbox table) applies cleanly.

## Definition of Done
- Green on trunk in every affected repo; pipelines use zero external systems.
- Boundary rule and contract-drift (Pact) gate enforced in CI, not by convention.
- EU/EEA region pinned in the Terraform skeleton (D-08).

## Open decisions
- Cloud provider/region for the managed cluster — thin ADR if not yet chosen (region must be EU/EEA regardless).

## Converges into
The `dream-backend` repo module structure, each repo's GitHub Actions workflows under `.github/workflows/`, the ArchUnit/Modulith rules, the Pact contracts, the `dream-infra` repo's Terraform modules, the Flyway baselines.
