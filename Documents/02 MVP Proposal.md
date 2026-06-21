# MVP Proposal — Minimum Viable Platform

_Draft · June 2026 · Companion to Target Architecture.md_

> **Premise.** The Architecture Concept is the target. This is day 1. We work backwards from the target and build the smallest platform that proves an end-to-end passenger journey, exercises every structural invariant the target depends on, and leaves each cut behind a seam the concept doc already names. The MVP extends through the ride itself, using the Lab (§8.3 of the concept) as the train — which puts the HIGH RISK sync seam on the critical path deliberately, sequenced so software retires the risk before hardware spends money on it.

---

## 1. The MVP in one sentence

**One passenger journey, end to end, in the Lab:** search → book → pay → ticket → board a scaled-model train → live facility status and berth controls in the passenger app → connectivity drops mid-trip and reconciles on reconnect → trip completes.

This is not just the commercial spine. It is the commercial spine **plus** the offline-first spine — the two things the target architecture cannot exist without — proven against physical hardware years before rolling stock arrives. It also covers all four priority scenarios in §8.3 of the concept, and doubles as the demo/investor asset D-19 anticipates.

## 2. Why ride, not just sell

A sell-only MVP — the obvious smaller alternative — has a structural weakness: the commercial core never touches the platform's biggest unprotected risk. Booking, payment, ticketing are online, cloud-side, strongly consistent — so a sell-only MVP starves the offline conflict-resolution model and the edge↔backend sync protocol (§7, HIGH RISK) of attention until the operator app forces the issue, roughly when real hardware arrives. The Lab inverts that: the ride experience *is* the forcing function. Every offline-first guarantee in the platform gets exercised by the MVP itself, on the physical instrument the concept doc designates for exactly this purpose (D-19).

The cost is real — this MVP is bigger than a sell-only one. The mitigation is sequencing (§6): the sync decision is made on rungs 1–2 (pure software) before the rig consumes it, so hardware never waits on an unsettled design.

## 3. What we build

### 3.1 Commercial spine

- **One modular monolith** (D-01's blessed starting shape). Postgres, one schema per domain, four modules: `trips`, `inventory`, `orders/ticketing`, `payments`. Hard module boundaries enforced in CI — no cross-schema queries, ever. The module map *is* the future service map.
- **No Kafka/NATS.** Transactional outbox (D-02) + Postgres-backed relay to idempotent in-process consumers is a durable, at-least-once bus. D-01 fixes the pattern, not the broker.
- **No separate BFF layer.** The monolith's API is the Passenger BFF. Kept invariant (D-05/D-06): external tokens validate and terminate at the API edge; only principal + realm claims travel inward.
- **One IdP product, realm-per-audience from day 1** (Keycloak realms map onto D-05). Passenger realm live (email/password + Google/Apple). **Operator realm now also live, minimally** — the ride needs a conductor (§3.3). The operator app caches a **trip-duration session on the device** (token + operator-realm JWKS) so a conductor can authenticate and validate fully offline (§3.3). eIDAS, HR provisioning flows, full MFA policy: later.
- **PSP in test mode behind the thin abstraction.** Nobody pays real money to ride a model train; merchant onboarding and real settlement are deferred to commercial launch. The integration is real, the money is not. Single currency (route is all-EUR — see pushbacks, §5).
- **Strong consistency where D-11 demands it** — inventory holds and ticketing as Postgres transactions inside one module. Drawn now because the boundary is brutal to retrofit.
- **Static fare table.** Yield engine later, behind the pricing interface.
- **Coarse internal trip-state.** The `trips` module carries a simple lifecycle status — `scheduled → boarding → en route → completed` (plus `cancelled`) — advanced manually from the thin admin and exposed read-only on the API. It is **internally driven, not IM running information** (the IM adapters stay cut, §4) — no delays, platforms, or forecasts, just where the trip is in its own lifecycle. The passenger app queries it; onboard it shows last-known state when offline. One enum column and a guarded transition; nothing more.
- **Thin back-office admin** in the same app: trips, consists, inventory, manifest. Includes the `paths` table + D-12 scheduling gate (one FK and a check constraint).

### 3.2 Offline-first spine

- **Contracts first (D-18).** Machine-readable contracts — OpenAPI for APIs, JSON-schema device shadows for IoT, AsyncAPI for sync messages — are the source of truth from which clients, fakes, and the conformance suite are generated. This precedes everything below.
- **Conformance suite (§8.2) as the keystone.** Every train surrogate — in-memory fake, software twin, scaled rig — must pass it. Built alongside the contracts, before any rig hardware is ordered.
- **Rungs 1–2 before rung 3.** The in-memory simulator and the software fleet twin (connectivity dropout, fault injection, concurrent manifest edits offline vs cloud-side mutations) are where the conflict-resolution model and sync protocol are *chosen*. Exit criterion: a written decision record. Only then does the rig implement the settled design.
- **Train Edge Gateway, production-real.** Small-form-factor x86/ARM running the same containers as production (D-19's no-divergence guardrail). Hosts the device-shadow store, the local event log, and the sync endpoint.
- **Device-shadow IoT layer (D-17), first real implementation.** Pluggable adapter contract; Pi-class sensors/actuators are throwaway, the adapter contract is not. Facility state real-time only, never persisted — the privacy boundary holds even in the Lab.
- **Scaled-model fleet rig (rung 3).** Model-train units conforming to *our* shadow contract — not emulating any real train protocol (we don't know it yet; D-19 is explicit on this). Value: physical movement, real BLE flakiness, multi-unit concurrency, actuator feedback.
- **Berth slice, not a full cabin.** One physical berth control panel (light, temperature, privacy lock) wired to the rig's edge gateway covers the night-trip scenario. The full cabin prototype (rung 4) comes later as a fit-and-finish and demo investment; the MVP needs the control loop, not the furniture.

### 3.3 Minimal operator surface

The ride needs a conductor. Smallest possible cut of the Operator App (D-10: Android, corporate device):

- Ticket validation (QR), **edge-first** with store-and-forward — see the connectivity matrix below.
- Mark cleaning task complete → ops/admin view updates (priority scenario 3).
- Nothing else. Rosters, messaging, meal queues, incident reporting: later.

**Ticket validation is edge-first and degrades across four connectivity states.** On board, the app talks to the **edge first whenever it is reachable**, falling back to the backend only when it is not; every validation records an event that reconciles, so a ticket can't be quietly used twice. Routing the on-board fleet through the single edge node also keeps all on-board devices consistent with each other in real time. Precedence: **edge → backend → device**.

| Edge | Internet | Validates against | Background sync |
|---|---|---|---|
| up | up | Edge (primary) | edge → backend continuously |
| up | down | Edge (primary) | edge → backend when uplink returns |
| down | up | Backend (fallback) | backend → edge once the edge reconnects |
| down | down | Device (pre-loaded manifest) | device → edge (or backend), whichever returns first |

Anything not yet synced to the backend is **optimistic**: acceptance is local and the source of truth catches up, so double-scan detection is the conflict case the settled model (§3.2) must handle. This is the precise sense in which ticketing's strong consistency (D-11) holds *at the backend* while on-board validation is accept-and-reconcile. The third row — backend-direct, then **back-sync to the edge** — is first-class, not an edge case: it is how an off-board or edge-down validation reaches the on-board queue. Nor is the fourth row a degraded afterthought: the operator app **pre-downloads the full manifest and validation rules at trip start**, so the device is a self-sufficient offline validator — the availability floor that keeps a conductor working with neither edge nor cloud. (It also caches a trip-duration operator session for offline auth, §3.1.)

This is deliberately the first consumer of the sync foundation — the operator app is where a wrong conflict model would hurt most, so it rides the settled design from its first line of code.

## 4. What we cut — and the return path for each

| Cut | Why safe | Returns as |
|---|---|---|
| Mobile passenger apps (native) | Responsive web reaches the edge gateway over the Lab Wi-Fi just as natively | Native apps over the same API/contracts |
| Distribution & Retail API / OSDM (D-20) | First-party proves the product | A surface over the same inventory/pricing modules |
| Loyalty (D-14) | Greenfield, no coupling | New module; only rule today: nobody stores a points balance |
| Data Warehouse & analytics | No volume | Read replica + Metabase, then event-driven ingestion |
| IM adapters, TIS, TAF/TAP (D-16) | No trains on real track | Adapter-per-IM behind the unified operational-data model |
| Split payment / hospitality flow (§5.2) | No galley; meal *ordering* can be faked later against the same order module | Saga over orders + payments, **plus the edge store-and-forward broker for in-trip offline orders** (§6.3–6.4 target) — rides the same sync/conflict seam the operator app already exercises |
| Real PSP settlement / merchant onboarding | No real money in the Lab | Flip from test mode at commercial launch |
| Full cabin prototype (rung 4) | Berth slice covers the control loop | Built when demo/UX value justifies it |
| Satellite uplink, mesh | Lab has Wi-Fi; dropout is *injected*, which is better for testing | Connectivity strategy, future phase |
| Multi-currency | Route is all-EUR | Config + PSP capability with first non-EUR market |
| Multi-tenancy machinery (D-09) | One logical country | A `tenant/country` column on the right tables **now**; nothing else |
| PNR | Already deferred (D-08) | Per-market activation behind its interface |

**Not negotiable** — retrofit cost is brutal: module/schema boundaries; outbox + idempotent consumers; token termination at the edge with realm claims; EU/EEA-only residency; accessibility baseline in the booking UI (TSI PRM, D-08); machine-readable contracts + conformance suite; edge gateway running production images.

## 5. Pushbacks on the target architecture

1. **Multi-currency "from day one" (D-07) is scope without a customer.** The launch route is entirely eurozone. Keep the abstraction; drop the requirement until a non-EUR market exists.
2. **Cloud-agnostic + GitOps ceremony on day 1 (D-03) is the doc's biggest hidden infrastructure cost.** One EU-region managed cluster, containers, plain CD. Portability is preserved by OSS-on-k8s; ArgoCD/Flux arrive with the second deployable and a bigger team.
3. **GraphQL-per-surface (D-04) has no work to do yet.** With one web surface, the monolith's API is the BFF. The Operator App is the second surface — when it grows beyond two screens, that's the moment to introduce a real BFF split.
4. **Guardrail to hold ourselves to:** a Lab-centred MVP can drift into a demo-driven side project — the exact failure D-19 warns about. The conformance suite is the defence: any rig component that stops passing it gets fixed or retired, and the edge gateway never runs anything but production images.

None of these reverse a target decision; each defers cost to where it earns its keep.

## 6. Sequencing

The two spines run in parallel; the sync decision gates hardware, not the other way round.

| Phase | Scope | Exit criterion |
|---|---|---|
| **M0** (first weeks) | Contracts (OpenAPI / AsyncAPI / shadow schemas), conformance suite skeleton, rung 1 in-memory sim | `make train` spins up a simulated train locally; CI green with zero external systems |
| **M1a** (≈ Q1–Q2) | Commercial spine: monolith, booking slice, PSP test mode, admin, paths gate | A stranger books a (test) trip and receives a ticket |
| **M1b** (parallel, ≈ Q1–Q2) | Rung 2 software fleet twin; dropout/fault injection; **sync protocol + conflict model decided** | Decision record written; twin passes conformance + scenario 4 |
| **M2** (≈ Q2–Q3) | Edge gateway on settled design; device-shadow adapter; rig + berth slice hardware; minimal operator app | Rig passes the conformance suite |
| **M3** (≈ Q3–Q4) | Integration: the full Lab ride | **The demo:** book on the web, scan the QR at the rig, watch the trip advance scheduled → en route → completed, watch toilet status flip live, set the berth light from the app, pull the plug mid-trip and reconcile on reconnect — validating a ticket in each of the four connectivity states (both, edge-only, backend-only, fully offline) |

Headcount: roughly six to eight engineers across both spines (edge/IoT/firmware skewed on the offline side). The commercial spine alone is a four-to-six-engineer, one-to-two-quarter effort; the ride is what the increment buys.

---

_This document proposes; the Architecture Concept disposes. Anything here that contradicts a D-NN is a pushback to be resolved, not a silent fork._
