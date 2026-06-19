# Project & Budget — Technology Domain

_Draft · June 2026 · Companion to Target Architecture.md, MVP Proposal.md, MVP Technical Choices.md_

> **Purpose.** A Principal-Engineer view of the technology programme over a three-year horizon: how tight the timeline is, the minimum team to deliver it, where to spend beyond the minimum to de-risk, and the buy-vs-build calls. **Scope is the technology domain only** — software development, the Lab, and production rollout once trains are available. Rolling-stock procurement, regulatory authorisation (ERA / SSC), path allocation, and the commercial/operations functions are owned elsewhere and treated here as external dependencies, not deliverables.

---

## 1. Scope boundary

**In scope.** The platform software (the eight surfaces, backend domains, edge, analytics — Target Architecture §4); the Lab (scaled-model rig + cabin prototype, §8.3); the contract-first toolchain and conformance suite (§8.2); production hardening and rollout; and the **rung-5 real-train integration** — the vendor commissioning handover and the read-only TCMS diagnostics path (D-15) — owned as a seam, not as train-building.

**Out of scope (owned elsewhere, consumed as dependencies).** Rolling-stock procurement and homologation; ERA vehicle authorisation and the Single Safety Certificate (D-13); path allocation and IM access agreements (D-12); commercial, finance, and operations staffing. Rail-operations and regulatory expertise is **not our headcount but is a hard dependency** — see §3.

---

## 2. Timeline: three years

**The decisive property: contract-first decouples our schedule from the train.** Because every external dependency ships with a stable internal contract and a substitutable simulator (D-18), nothing we build waits on a real train, a path, or a certificate. The real train is rung 5 — a final validation against an interface we will have tested against for three years. Our timeline is therefore **self-determined**, a function of three things we control: scope breadth, team size, and when the sync risk is retired.

**Assessment.** Three years is **comfortable for the core** — the commercial spine, the offline-first spine, the Lab, and production hardening. It becomes **tight only if the full eight-surface target** (OSDM distribution, the analytics warehouse, the full hospitality domain, native mobile) is held in-scope for the same window with a minimum team. The resolution is the discipline the MVP Proposal already shows: phase ruthlessly, keep cut surfaces behind named seams, let breadth arrive after the spine is proven.

**The one schedule risk that can break it** is the offline conflict-resolution / edge-sync seam (§7, HIGH RISK) — the only research-grade unknown, and the gate on everything offline-first. It must be retired early, on rungs 1–2, before hardware consumes it (MVP Proposal §6). If it slips, the back half of the plan compresses badly.

---

## 3. Minimum team

A credible minimum at full ramp is **~11–13 engineers plus a thin specialist layer**, reached by ramping — not hiring all at once (Year-1 MVP runs at the six-to-eight figure in MVP Proposal §6).

| Role | Min count | Why |
|---|---|---|
| Backend / domain (Java/Spring) | 5–6 | The monolith decomposing into services across trips/inventory/pricing, ticketing, payments, path, fleet/compliance, loyalty, hospitality, integration hub; **owns the contract + conformance harness** (§8.2) as a team responsibility rather than a dedicated SDET |
| Edge / offline (Go) | 1–2 | Device shadows + the sync protocol; needs real distributed-systems depth, not a generalist |
| Web (React/TS) | 1 | PWA, booking UX, accessibility-strong (TSI PRM is a day-one CI gate, D-08) |
| Android (Kotlin) | 1 | Operator app (D-10) |
| Platform / SRE | 1–2 | k8s, Terraform, GitHub Actions, observability, EU/EEA residency (D-08) — non-negotiable |
| Embedded / IoT | 1 | The Lab — BLE, sensors/actuators, edge hardware, firmware, BOM. Cannot be staffed with web developers |

**Thin specialist layer (fractional or shared):** a security engineer (identity topology, payments, GDPR — §5.1) and accessibility-strong UX competence (whether in our org or adjacent).

**Secured dependency, not headcount:** guaranteed access to a **rail-operations / regulatory SME**. We consume this expertise rather than own it, but we must have it on tap — the failure mode is engineers modelling path gating, TAF/TAP, and fleet compliance plausibly and wrongly, and that doesn't care whose budget the SME sits on.

---

## 4. Where to spend beyond the minimum (de-risking)

Ranked by risk bought down, not by headcount:

1. **A second senior distributed-systems engineer on the sync seam.** The biggest internal risk currently rests on one or two people — bus-factor and depth risk at once. The single best de-risking euro.
2. **A real SRE/platform team (2–3) and a full-time security engineer.** "Rollout to production once trains are available" is explicitly ours; the gap between "runs in the Lab" and "runs a 24/7 revenue service" is wide, and an identity/payments/GDPR security miss is existential.
3. **A rung-5 integration owner.** When trains arrive, one person should own the commissioning handover and TCMS diagnostics bring-up (D-15) so it doesn't drag the whole team off the roadmap. The in-scope analogue of the train dependency: not building the train, owning the seam where the platform meets it.
4. **Engineering management.** At 13–16 people a principal who is also the only leader becomes the bottleneck.
5. **A payments-experienced engineer and an earlier data engineer.** The split base/hospitality saga (§5.2), multi-currency, and settlement/reconciliation are harder than they look; the warehouse/yield foundation underpins commercial pricing and is better laid early than scrambled late.
6. **A second embedded/hardware engineer** for the cabin prototype and rig, to de-risk the physical timeline.

---

## 5. Buy vs build

**Keep buying (commodity, never our moat):** identity (Keycloak), payments (Adyen/Stripe), Postgres, Kafka/NATS, k8s, the observability stack, BI tooling — all already chosen. Extend the same instinct to eIDAS/government-ID verification (via Keycloak brokering), KYC/fraud/tax-VAT/invoicing, MDM, timetable/journey-planning data feeds, the CRDT *library* (Automerge/Yjs — buy the primitive, build the protocol), and off-the-shelf Lab hardware.

**Keep building (the differentiation):** the offline-first edge + device-shadow IoT layer, the onboard hospitality/cabin experience, and the conformance suite / fidelity ladder. None of it is on the shelf, and it is where the product lives.

**One live call, not yet settled in the docs:**

- **The reservation/PSS core.** We currently build booking/inventory/ticketing. Our differentiation is not the reservation engine — it is the edge, the offline spine, and the cabin. There is a real case to buy or partner for commodity rail retail/PSS and build only the differentiated layer; the counter-argument is D-01's single inventory truth and OSDM control. Lean **build-but-thin**, but decide deliberately — it is the largest block of undifferentiated engineering in the plan.

---

## 6. Indicative three-year phasing

Maps onto the MVP sequencing (MVP Proposal §6) for Year 1, then breadth and production in Years 2–3.

| Phase | Focus | Team | Exit |
|---|---|---|---|
| **Year 1 — MVP (M0–M3)** | Contracts + conformance + rung-1 sim; commercial spine; rung-2 twin and **sync decision settled**; edge gateway + device-shadow + rig + berth slice; minimal operator app; the full Lab ride | ~6–8, ramping | The demo: book on web → scan at the rig → trip advances → facility status live → berth control → connectivity drop and reconcile |
| **Year 2 — Breadth + production-grade platform** | Service extraction along the module seams; full fleet/compliance and path workflow; hospitality and loyalty; Distribution API (OSDM); analytics warehouse; native mobile begins; full multi-realm identity; SRE + security hardening | ramp to ~11–13 + SRE/security/data/embedded depth | Revenue-grade platform running on contracts; second deployable extracted; production observability and on-call |
| **Year 3 — Production rollout + rung-5** | Real-train commissioning handover and TCMS integration; pre-trip validation against real hardware; performance, DR, operational readiness; scale-out | sustained ~13–16 with EM + rung-5 owner | Platform validated against a real train; ready to operate when trains, authorisation, and paths (owned elsewhere) land |

---

## 7. Indicative costs (back-of-envelope)

Rough order-of-magnitude only — enough to size the programme, not a budget. Assumptions:

- **Salaries:** 2026 Berlin gross market rates for the role mix in §3.
- **Fully-loaded employment cost ≈ gross × 1.20** (employer social-security contributions). Excludes facilities, equipment, and recruiting — add ~10–15% if those are carried here.
- **Headcount follows the ramp in §6** (Year-1 avg ~7, Year-2 avg ~12, Year-3 avg ~15). The blended cost per head rises year on year as senior/specialist roles (EM, security, SRE) are added.

**Payroll**

| Year | Avg headcount | Blended loaded cost / head | Payroll |
|---|---|---|---|
| 1 | ~7 | ~€110k | **~€0.8M** |
| 2 | ~12 | ~€112k | **~€1.35M** |
| 3 | ~15 | ~€116k | **~€1.75M** |
| | | **3-year payroll** | **~€3.9M** |

**Tooling & infrastructure.** The buy recommendation (§5) favours **OSS self-hosted** for the heavy components — Keycloak, Postgres, Kafka/NATS, Prometheus/Grafana, Metabase/Superset — so fixed SaaS *subscriptions* stay modest and the material recurring line is **cloud infrastructure**. Payment-processing fees (Adyen/Stripe) and per-verification identity/KYC costs are **transactional and revenue-linked** — ~€0 in test mode (Years 1–2), scaling with bookings thereafter — so they sit outside this fixed opex.

| Year | SaaS subscriptions | Cloud infra | Tooling + infra |
|---|---|---|---|
| 1 | ~€40k | ~€80k | **~€0.12M** |
| 2 | ~€60k | ~€200k | **~€0.26M** |
| 3 | ~€110k | ~€400k | **~€0.51M** |

_SaaS line:_ GitHub seats, MDM for operator devices, error-tracking/dev tooling, eIDAS/KYC verification (low pre-launch), and optional managed identity/observability at production scale. _Cloud infra:_ managed k8s plus compute/storage/network across dev/staging/prod in an EU region, growing to HA/DR at production.

**Programme total (excluding Lab hardware capex)**

| Year | Payroll | Tooling + infra | Total |
|---|---|---|---|
| 1 | €0.8M | €0.12M | **~€0.92M** |
| 2 | €1.35M | €0.26M | **~€1.61M** |
| 3 | €1.75M | €0.51M | **~€2.26M** |
| | | **3-year total** | **~€4.8M** |

**Not included:** Lab hardware capex (the scaled-model rig, Pi-class sensors/actuators, production-spec edge nodes, the berth panel — a one-off in the **low tens of thousands**, ~€30–60k, mostly Years 1–2); office/facilities; and the transactional payment/verification fees noted above.

---

## 8. Summary

Within the technology boundary, three years is realistic and largely under our own control — contract-first hands us a schedule that does not wait on trains, paths, or certificates. Protect it by phasing breadth behind seams, retiring the offline-sync risk early on software, and spending the first marginal euro on distributed-systems depth, then on the SRE/security maturity that production rollout demands. Buy the commodity, build the edge and the conformance ladder, and make the reservation-core call deliberately rather than by default.

---

_This document proposes within the technology scope; it consumes the train, regulatory, and path timelines as external dependencies. Anything here that contradicts a D-NN is a defect, not a decision._
