# MVP Technical Choices

_Draft · June 2026 · Companion to MVP Proposal.md and Target Architecture.md_

> **Purpose.** Records the concrete language/runtime choices for the MVP, one line of reasoning each. These are MVP commitments, not target-architecture decisions (D-NN) — they implement the architecture, they don't change it. Where the architecture leaves a choice open, this document closes it for day 1 and names what stays deferred.

---

## Summary

| Component | Choice | Notes |
|---|---|---|
| Web app | **React + TypeScript** | Responsive PWA; first-party passenger surface |
| Android app (Operator) | **Kotlin + Jetpack Compose** | Corporate-issued, MDM-managed (D-10) |
| Backend Services | **Java + Spring Boot** | Modular monolith (D-01); **Gradle** build |
| Database | **PostgreSQL** | One schema per domain (D-01) |
| Train Edge Gateway | **Go** | On-board offline node (§6.4) |
| Contracts | **OpenAPI** (+ AsyncAPI, JSON Schema) | Machine-readable source of truth (D-18) |
| Platform / CI-CD | **Terraform + GitLab CI** | One EU-region K8s cluster |
| Repository | **Multi-repo: one per surface/component + contracts + dream-orchestration** | Contracts shared via git submodule; dream-orchestration repo owns `make train` |
| Testing | Per stack — see below | Conformance suite is the keystone (§8.2) |

---

## Rationale

**Web app — React + TypeScript.** Chosen primarily for the accessibility mandate: TSI PRM compliance is non-negotiable from day one (D-08), and React has the deepest accessible-primitive ecosystem (React Aria / Radix) to build the booking and seat/berth-selection UI against. Delivered as an offline-capable PWA — service worker (Workbox) plus IndexedDB for the cached ticket/booking (stories P-7, P-11) — and reaching the Train Edge Gateway directly over Lab Wi-Fi for in-trip features. No GraphQL yet; the monolith's REST API is the BFF (D-04 pushback).

**Android app — Kotlin + Jetpack Compose.** D-10 fixes the Operator App as Android-only on managed devices, so there is no cross-platform tax to pay. Native Kotlin gives the best offline storage (Room), camera/QR handling (CameraX + ML Kit), and MDM integration. Scope is the minimal conductor + cleaner cut (MVP §3.3): offline ticket validation and cleaning-task completion.

**Backend Services — Java + Spring Boot (Gradle).** A modular monolith is D-01's blessed starting shape, and the Spring ecosystem implements its invariants directly: Spring Modulith enforces module boundaries in tests (the "no cross-schema queries" rule), Spring Security as an OAuth2 resource server terminates Keycloak tokens at the API edge (D-06), and the outbox/idempotency machinery (D-02) is well-trodden here. Java 21+ with virtual threads keeps the code imperative and blocking while scaling — no reactive complexity. Strong consistency lives inside the `orders/ticketing` and `payments` modules as Postgres transactions (D-11), with explicit SQL (jOOQ or Spring Data JDBC) on the money paths and JPA elsewhere. **Gradle** is the build tool across the repo so one toolchain spans the Java backend and the Kotlin app.

**Database — PostgreSQL.** One logical Postgres instance, one schema per domain module (D-01), with the transactional outbox table co-located with each module's state so publication and commit are atomic (D-02). Strongly-consistent inventory holds and ticketing run as Postgres transactions (D-11). Flyway for migrations. A `tenant/country` column goes on the right tables now; no other multi-tenancy machinery (MVP §4).

**Train Edge Gateway — Go.** The gateway is a stateful, resilient, long-running on-board node — local device-shadow store, local event log, offline ticket validation, the sync endpoint — not a stateless proxy. It is, however, **not performance-critical**: all safety-critical and movement control is out of scope (D-15), and the workload (facility status, berth controls, ticket scans) is light and soft-real-time. Go fits that profile well: robust, simple, excellent concurrency, container-native, with an embedded SQLite store and BLE handled through OS bindings. It runs the same production container images on the Lab rig as in production (D-19's no-divergence guardrail).

**Contracts — OpenAPI (+ AsyncAPI, JSON Schema).** Contract-first is a structural commitment (D-18): the machine-readable contracts are the source of truth from which clients, fakes, and the conformance suite are generated — not documentation written after the code. Three contract languages cover the three interaction styles the MVP has: **OpenAPI** for the REST APIs (backend, web, Android), **AsyncAPI** for the edge↔backend sync messages, and **JSON Schema** for the device shadows. `openapi-generator` produces server interfaces and typed clients; **Spectral** lints the specs in CI so they can't drift or degrade. This precedes the code it describes (MVP §3.2).

**Platform / CI-CD — Terraform + GitLab CI.** Per the MVP's deliberate de-scoping of day-1 platform ceremony (§5 pushback #2): one EU-region managed Kubernetes cluster, containers, plain continuous delivery now — ArgoCD/Flux GitOps arrive with the second deployable and a bigger team. **Terraform** is the infrastructure-as-code tool, keeping the estate cloud-portable and honouring the cloud-agnostic intent (D-03) without committing to a proprietary control plane. **GitLab CI** runs the pipelines: build, test, container build, and deploy, with the M0 gate that CI is green with zero external systems (`make train`). EU/EEA-only residency is enforced at the cluster/region level (D-08). Observability is wired from the first service — OpenTelemetry traces, Prometheus + Grafana metrics, structured logs — so it is never a retrofit.

---

## Repository approach

**Day 1: a repository per product surface/component, plus a shared dream-contracts repo and a dream-orchestration repo.** Each stack lives at the root of its own repository, on its own trunk, with its own self-contained build and pinned toolchain (S-00):

| Repo | Stack / role | Build |
|---|---|---|
| `dream-backend` | Java/Spring Boot modular monolith | Gradle |
| `dream-passenger-web-app` | React + TS PWA | npm / Vite |
| `dream-operator-android-app` | Kotlin + Jetpack Compose | Gradle |
| `dream-edge` | Go Train Edge Gateway | Go toolchain |
| `dream-contracts` | OpenAPI / AsyncAPI / JSON-Schema — the source of truth | Spectral lint + `openapi-generator` |
| `dream-infra` | Terraform, Keycloak realm exports, cluster config | `terraform validate` |
| `dream-orchestration` | `make train` / `make verify`, docker-compose, the conformance harness, the rung-1 sim, cross-repo scripts | Make + Docker |

This is a change from the prior plan, which started in a single loosely-coupled monorepo and named multi-repo (edge first) only as a deferred seam. We now take the split up-front. The cost the monorepo was avoiding does not disappear — it moves to two specific mechanisms, below.

**Contracts are shared by git submodule, pinned at a SHA.** The `dream-contracts` repo is the single source of truth (D-18); each consuming repo (backend, web, Android, edge, dream-orchestration) embeds it as a **git submodule pinned at an explicit commit** and runs `openapi-generator` against the pinned specs in its own build. A contract change is one commit in `dream-contracts`; each consumer adopts it by bumping its submodule pointer — a small, reviewable, atomic step per repo rather than a publish-and-version-bump dance through a package registry. This keeps codegen reproducible (every repo builds against an exact, recorded contracts SHA) without a publishing pipeline on day 1.

**Drift — the failure mode §8.2 calls the killer — is held off by two gates, not by one-checkout convenience.** First, **consumer-driven contract tests (Pact)** run in each repo's CI against the pinned contracts, so a consumer that has fallen behind the contract fails its own build. Second, the **shared conformance suite (§8.2) lives in the `dream-orchestration` repo** and runs every train surrogate (in-memory fake, software twin, scaled rig) over the wire against the same contracts — the rung-agnostic harness that stops the rungs diverging. The compiled-module safety net the monorepo gave for free is replaced explicitly by Pact + the conformance harness (already the plan in the Testing section).

**The M0 one-command bring-up lives in the dream-orchestration repo and checks out its siblings.** `make train` (and `make verify`) in `dream-orchestration` clones/updates each sibling repo at a **pinned SHA**, builds them, and brings up the full simulated train locally with zero external systems — preserving the M0 gate across repos. Each repo also keeps its own minimal CI (build + test + Pact); the dream-orchestration pipeline is the cross-repo green heartbeat that the M0 exit criterion depends on. (Whether bring-up later shifts to pulling each repo's published container images instead of building from source is a `dream-orchestration` repo decision, deferred.)

**The decoupling that matters is still enforced in code, not merely by repo topology.** Database-per-service ownership (D-01), the Spring Modulith + ArchUnit boundary rules inside `dream-backend` (S-01), and the contract interfaces remain the real seams. Repo boundaries now add release-cadence and access-control isolation on top — useful, but not a substitute for the in-code seams.

**What each repo's separation buys, and the open items it creates:**

- **`dream-edge` gets firmware-grade release discipline from day 1** — on-board, hardened, deployed to physical devices rarely and deliberately, never sharing a release train with continuously-shipped web. The reserved Rust/firmware-unification path (see *Deferred*) sits naturally behind this boundary.
- **`dream-operator-android-app`** aligns to the Play/MDM release lifecycle without dragging the backend's cadence.
- **`dream-backend` is one repo, still one deployable** (modular monolith, D-01). When modules extract into independent services later, whether each service also gets its own repo is a separate, team-topology-driven decision made then — the repo split today is at the *surface* boundary, not the backend-module boundary.
- **`dream-contracts` as its own repo** is also what later enables scoped external access (OSDM retailers via the Distribution API, IoT/hardware vendors) without restructuring.
- **Submodule discipline is the new tax:** every consumer must keep its `dream-contracts` pointer current, and CI must fail a stale one (via Pact). This is the cost we accept in exchange for surface isolation up front.

The message in one line: **one repo per surface/component from day 1, with `dream-contracts` shared by pinned submodule and a `dream-orchestration` repo owning `make train` and the conformance harness — so surfaces get independent cadence and access scope immediately, while Pact + the conformance suite (not a single checkout) keep contracts and the rungs from drifting.**

---

## Testing

The keystone is the **shared conformance suite** (§8.2) that every train surrogate must pass — written as a black-box, over-the-wire harness so the same tests hit the in-memory fake, the software twin, and the scaled rig alike. Below it, each stack has its own test tooling.

| Stack | Unit / integration | End-to-end / UI | Other |
|---|---|---|---|
| Backend (Java/Spring) | **JUnit 5**, AssertJ, Mockito, Spring Boot Test | MockMvc / REST Assured | **Testcontainers** (real Postgres), **ArchUnit** (module-boundary enforcement), **WireMock** (external stubs) |
| Web (React/TS) | **Vitest** + React Testing Library | **Playwright** | **axe-core** (accessibility — TSI PRM gate), **MSW** (API mocking) |
| Android (Kotlin) | **JUnit 5**, MockK, Turbine, Robolectric | **Compose UI test** (Espresso underneath) | screenshot tests (Roborazzi) optional |
| Edge gateway (Go) | Go `testing` + **testify** | scenario tests via the conformance suite | **Testcontainers-go** for integration |
| Contracts | spec validation (**Spectral**) | — | **Pact** (consumer-driven contract tests across provider/consumer), **Schemathesis** (property-based API tests from OpenAPI) |
| Platform | `terraform validate` / **tflint** | smoke tests post-deploy | **Trivy** (container/IaC scanning) |

Two non-negotiables to call out: **accessibility testing (axe-core) is a CI gate on the web app**, not a manual afterthought (D-08); and **contract tests (Pact)** guard the boundaries that lost their compiled-module safety net once the stack went polyglot.

---

## Deferred / not chosen yet

- **Edge sync language for the CRDT engine.** Go for the gateway is settled; if the conflict-resolution decision (§7, HIGH RISK) lands on CRDTs, the Automerge binding choice is revisited then. The decision itself is made on rungs 1–2 first (MVP §6).
- **Event broker.** No Kafka/NATS in the MVP — transactional outbox + Postgres-backed relay to in-process consumers (MVP §3.1). Broker is added with the second deployable.
- **BFF split / GraphQL.** BFF stays a module inside the monolith; first extraction is the Distribution API, post-MVP (D-20, D-04 pushback).
- **PSP.** Adyen integration runs in test mode behind the abstraction (MVP §3.1); real settlement at commercial launch.
- **Edge runtime alternative.** Rust was weighed for the gateway and reserved for the case where firmware unification or maximal robustness justifies it; not needed for the MVP workload.

---

## Resulting language map

Four languages, each on its strongest ground: **TypeScript** (web), **Kotlin** (Android), **Java** (backend), **Go** (edge). Java and Kotlin share the JVM toolchain and Gradle, so the effective load is lighter than four independent ecosystems. Keycloak (identity) and PostgreSQL (data) are shared infrastructure across all of them.

---

_Version-stamp updates when a choice changes. Anything here that contradicts a D-NN is a defect in this document, not a decision._
