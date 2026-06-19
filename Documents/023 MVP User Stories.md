# User Stories — MVP (Minimum Viable Platform)

_Draft · June 2026 · Companion to Target Architecture.md and MVP Proposal.md_

> **Lens.** Written from a UX/UI design + research perspective. Each story is framed by the value to the person, not the implementation. Acceptance criteria are kept to what makes the story a **must-have for the MVP** — nothing aspirational.

---

## Scope boundary (what "MVP" means here)

The MVP is **one passenger journey, end to end, in the Lab**: search → book → pay → ticket → board the scaled-model train → live facility status + berth controls in the passenger app → connectivity drops and reconciles → trip completes (MVP Proposal §1).

A use case earns a place below only if **that single journey cannot happen without it**. Everything the MVP Proposal cuts (§4) is out — and listed explicitly at the end so the boundary is visible, not implied.

**Personas considered:** Passenger, Operator (Operations/Admin, Conductor, Cleaner), Corporate/Analyst.

**Surfaces in the MVP:** responsive **passenger web** (native apps are cut — §4), a **thin back-office admin** inside the monolith, and a **minimal Operator App** on a corporate Android device (§3.3). No native passenger app, no GraphQL BFF split, no separate ops web app.

---

## Cross-cutting acceptance criteria (apply to every relevant story)

These are the MVP Proposal's "not negotiable" items (§4). Rather than repeat them per story, they hold throughout:

- **Accessibility (TSI PRM):** every passenger-facing booking screen meets the accessibility baseline — full keyboard operation, screen-reader labels, sufficient contrast, visible focus. This is brutal to retrofit and is treated as a definition-of-done gate, not a later polish.
- **Offline behaviour is a first-class state, not an error.** Wherever a story runs onboard, the UI shows connection status honestly, keeps edge-served features working, degrades cloud-only features gracefully, and reconciles on reconnect.
- **Realm isolation:** passenger and operator credentials never cross; external tokens terminate at the edge, only principal + realm claims travel inward.
- **Facility state is real-time only, never persisted** — a passenger-privacy boundary that holds even in the Lab.
- **Single currency (EUR)** and **static fares** — multi-currency and the yield engine are out.

---

## Passenger (responsive web)

### Discovery & booking

**P-1 — Search for a trip**
As a passenger, I want to search for a service by origin, destination and date, so that I can find a journey that fits my plans.
- Results show departure/arrival times, duration and a price from the static fare table.
- Single launch route; all-EUR.

**P-2 — See availability and price**
As a passenger, I want to see what accommodation is available and what it costs, so that I can decide what to book.
- Shows day-seat and night-berth options for the service.
- Availability reflects the strongly-consistent inventory truth — I am never shown a seat/berth that is already taken.

**P-3 — Select a specific seat or berth**
As a passenger, I want to choose my exact seat or berth from a layout, so that I get the accommodation I want.
- Selection UI meets the TSI PRM accessibility baseline (non-negotiable).
- A short-lived hold is placed on the chosen inventory while I complete booking.

**P-4 — Create an account / sign in**
As a passenger, I want to register or sign in with email/password, Google or Apple, so that I can hold a booking and retrieve my ticket later.
- Passenger realm only; MFA optional at this stage.

**P-5 — Pay for my booking**
As a passenger, I want to pay for my ticket and have the booking confirmed, so that my place is secured.
- Payment runs through the PSP in **test mode** behind the abstraction (real integration, no real money).
- The booking is confirmed only on a successful (test) charge; payment and ticketing are strongly consistent — no double-charge, no double-book.

**P-6 — Receive a digital ticket**
As a passenger, I want a ticket with a scannable QR code after I pay, so that I can board the train.
- QR issued against the confirmed booking and linked to the manifest entry.

**P-7 — Access my ticket without a connection**
As a passenger, I want my ticket and booking details available even with no signal, so that I can board when connectivity is poor.
- Ticket/booking cached locally; the QR renders and validates offline.

### On board (in the Lab)

**P-8 — See live facility availability**
As a passenger on board, I want to see real-time facility status (e.g. toilet occupied/free), so that I know when it is free to use.
- Served live via the edge gateway; **never persisted**.
- Covers Lab priority scenario 2.

**P-9 — Control my berth environment**
As a night-trip passenger, I want to set my berth's light, temperature and privacy lock from the app, so that I can make my space comfortable and private.
- Round trip: app → edge gateway → device shadow → physical actuator → sensor feedback → app update.
- Covers Lab priority scenario 1 (berth slice — one physical panel).

**P-10 — Stay in control when connectivity drops**
As a passenger, I want edge-served features (berth controls, facility status) to keep working when the train loses uplink, and cloud features to degrade gracefully with a clear status, so that the experience holds together mid-trip.
- App reaches the edge gateway directly over Lab Wi-Fi; connection state is visible.
- State reconciles correctly on reconnect.
- Covers Lab priority scenario 4 (the HIGH RISK sync seam, exercised end to end).

**P-11 — See the state of my trip**
As a passenger, I want to see where my trip is in its lifecycle (scheduled, boarding, en route, completed), so that I know what's happening with my journey before and during the ride.
- Coarse internal trip-state only, **internally driven** — not IM running information (no delays, platforms, or forecasts; that adapter is cut).
- Read-only query; last-known state shown when offline onboard.

---

## Operator

### Operations / Admin (thin back-office, inside the monolith)

**O-1 — Create and schedule a trip**
As an operations admin, I want to create a trip with its route, date/time and day-or-night configuration, so that it can be sold and run.

**O-2 — Configure the consist and inventory**
As an operations admin, I want to define the train composition and its sellable inventory (seats, berths), so that passengers can book real accommodation.
- Night config exposes berths; day config exposes seats.

**O-3 — Maintain the fare table**
As an operations admin, I want to set prices in a static fare table, so that trips have fares to sell against.
- Static only; no yield/pricing engine in the MVP.

**O-4 — Block scheduling without a held path**
As an operations admin, I want the system to refuse to schedule a trip unless a valid held path exists for that date, so that we never commit to running a service we have no right to operate.
- Enforces the D-12 gate (paths table + FK + check constraint).
- Path state is recorded manually in the MVP — no IM-portal/PCS integration yet.

**O-5 — View the passenger manifest**
As an operations admin, I want to see the manifest for a trip, so that I know who is booked and can support the onboard crew.
- Manifest derived from confirmed bookings.

**O-6 — Advance the trip state**
As an operations admin, I want to move a trip through its lifecycle (scheduled → boarding → en route → completed, or cancelled), so that passengers see accurate status.
- Single guarded transition from the thin admin; published via the outbox so the passenger query (P-11) reflects it.
- Simplest possible: manual transition, no IM input, no automatic detection. (Crew-triggered transitions can come later.)

### Conductor (minimal Operator App — corporate Android)

**O-7 — Sign in and stay authenticated offline**
As a conductor, I want to sign in on my corporate device and remain authenticated for the whole trip without a live connection, so that I can work onboard regardless of signal.
- Operator realm (minimal); tokens cached on the edge gateway, valid for the full trip duration; MDM-managed device.

**O-8 — Validate tickets offline**
As a conductor, I want to scan a passenger's QR ticket and validate it against the edge gateway even with no uplink, so that I can check passengers in onboard.
- Validation runs against the edge gateway; works fully offline and syncs to the cloud on reconnect.
- Validation is idempotent (safe replays after sync).
- This surface is the **first consumer of the settled sync design** (MVP Proposal §3.3).

### Cleaner (minimal Operator App)

**O-9 — Mark a cleaning task complete offline**
As a cleaner, I want to mark a cleaning zone or task as complete on my device, so that operations can see turnaround progress.
- Works offline and syncs; the ops/admin view updates once reconciled.
- Covers Lab priority scenario 3.

---

## Corporate / Analyst

**No must-have user stories in the MVP — and this is deliberate.**

The Data Warehouse and analytics surface is explicitly cut for the MVP (MVP Proposal §4): there is no data volume to justify it, and the Company IdP / read-only DW boundary brings no value to the single Lab journey. As a researcher I'd flag rather than invent: forcing an analyst story here would breach the "must-have only" rule.

The persona is **served indirectly** — every booking, validation, sync event and cleaning completion produced by the stories above is generated through the outbox, so the behavioural dataset accrues from day one even though nobody queries it yet. The return path (§4) is a read replica + Metabase, then event-driven ingestion, with no schema rework required.

One adjacent corporate need *is* met by the MVP, but it is a stakeholder outcome rather than a software use case with a UI: the end-to-end Lab ride doubles as the **investor/demo asset** (D-19) — book on the web, scan at the rig, watch facility status flip, set the berth light, pull the plug mid-trip, reconcile on reconnect.

---

## Explicitly out of scope (not user stories — boundary markers)

Listed so the cut line is visible. Each returns behind a seam the architecture already names (MVP Proposal §4):

- Native passenger mobile apps; push notifications.
- Third-party distribution / OSDM retail API.
- Loyalty (accrual, tiers, redemption).
- Live IM running information — delay/forecast/disruption (IM adapters, TIS, TAF/TAP all cut). The coarse **internal** trip-state indicator (P-11) is in scope; **external IM-derived live status is out**.
- Hospitality / meal ordering and the split base/hospitality payment flow.
- Real PSP settlement and merchant onboarding.
- Full cabin prototype (rung 4) beyond the single berth slice.
- Rosters, crew messaging, meal queues, incident/fault reporting in the Operator App.
- Multi-currency, dynamic pricing/yield.
- Multi-tenant machinery (a `tenant/country` column is added now; nothing more).
- PNR collection.
- Self-service booking changes/cancellations/refunds.

---

_Anything that contradicts a D-NN in the Target Architecture is a pushback to resolve, not a silent fork (MVP Proposal closing note)._
