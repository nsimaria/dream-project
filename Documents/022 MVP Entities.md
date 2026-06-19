# Entities — MVP (Minimum Viable Platform)

_Draft · June 2026 · Companion to Target Architecture.md, MVP Proposal.md, User Stories.md_

> **Purpose.** A first-pass **catalogue** of the domain entities the MVP walking skeleton touches — _search → book → pay → ticket → board → live facility/berth → connectivity drop & reconcile → trip completes_ (MVP Proposal §1). Each entry is a one-line purpose plus its key relationships. **No attributes yet** — fields, types, and constraints come in a later pass once the set and the boundaries are agreed.

---

## Scope & reading rules

- **MVP only.** An entity earns a place only if the single Lab journey cannot happen without it. Everything cut in MVP Proposal §4 — loyalty, hospitality/meals, IM running information, the Distribution API, the data warehouse, multi-currency, full fleet/maintenance — is **out**, and noted at the end so the cut line stays visible.
- **Organised by owning module.** The grouping mirrors the four MVP backend modules (`trips`, `inventory`, `orders/ticketing`, `payments`) plus identity and the edge, each the **sole source of truth for its own data** (D-01). No module reads another's schema.
- **Cross-domain links are references, not foreign keys across schemas.** Where one domain points at another (e.g. a `Booking` at a `Trip`), it holds an **ID and consumes the other's events** (D-01/D-02) — never a cross-schema join. Those links are written below as _Refs:_ to keep that honest.
- **Derived vs owned.** Some entries are **projections / read models** (e.g. the manifest, the seat-map), rebuilt from events rather than authored. They're flagged so we don't mistake a view for a system of record.

---

## Trips (`trips` module)

**Trip** — a scheduled journey on a date with a day-or-night configuration, carrying its own coarse lifecycle status `scheduled → boarding → en route → completed` (plus `cancelled`), advanced manually and internally — not from IM data (O-1, O-6, P-11). _Refs:_ a `Route`, a `Consist`, a held `Path`; owns a `Manifest`; exposes sellable `Inventory`.

**Route** — the origin→destination service definition (with intermediate stops) a `Trip` runs on; single launch route in the MVP (P-1). _Refs:_ an ordered set of `Station`s.

**Station** — a stop on the route (Berlin, Cologne, Brussels, Paris); reference data for search and timetabling (P-1). _Refs:_ appears in `Route`.

**Fare** — a static fare-table entry giving a price for an accommodation type on a service; no yield/pricing engine (O-3, P-1, P-2). _Refs:_ priced against `Trip` / accommodation type.

> Trip lifecycle status is modelled as a guarded enum transition on `Trip`, not a separate entity; each transition emits a domain event via the outbox so the passenger query (P-11) reflects it.

## Path & slot (minimal — `paths` table, D-12 gate)

**Path (Slot)** — a held operating right for a date and IM; its presence and validity **hard-gate trip scheduling** (O-4, D-12). Recorded manually in the MVP — no PCS/IM-portal integration. _Refs:_ gates a `Trip` (FK + check constraint).

## Fleet / consist (minimal — composition only)

**Consist** — the ordered train composition for a trip, in day-config (seats) or night-config (berths), defining what inventory exists (O-2). Modelled as a **dated instance bound to a single trip**, not a reusable template: the same physical carriages re-marshalled on another date form a distinct consist. _Refs:_ assigned to exactly one `Trip`; composed of `Carriage`s. _(Full fleet registry, maintenance, and authorisation gates are cut — MVP Proposal §4.)_

**Carriage / Unit** — a vehicle within a consist that carries seats or berths. _Refs:_ belongs to a `Consist`; lays out `InventoryUnit`s.

## Inventory (`inventory` module — strongly consistent)

**InventoryUnit (Accommodation)** — a single sellable seat or berth on a specific trip; availability is the **strongly-consistent truth** a passenger is shown (P-2, P-3, D-11). Modelled **polymorphically** — one entity with a type (seat / berth / suite-later) — so holds, seat-map, and booking logic stay generic across day/night config. _Refs:_ laid out in a `Carriage` of the trip's `Consist`; subject to `Hold` and `Booking`.

**Hold** — a short-lived reservation on an `InventoryUnit` while a passenger completes booking, preventing double-sell (P-3). _Refs:_ on one `InventoryUnit`, for a booking session; expires.

> **Seat/berth map** is a **projection** over the consist + inventory + holds, served to the selection UI (P-3); it is a read model, not an owned entity.

## Orders & ticketing (`orders/ticketing` module — strongly consistent)

**Booking (Order)** — a passenger's purchase of one or more accommodations on a trip, **confirmed only on a successful (test) charge** (P-5, D-11). _Refs:_ a `Passenger` (by principal claim); a `Trip`; the `InventoryUnit`(s) it consumes; a `Payment`; issues `Ticket`(s).

**Ticket** — issued **one per booked accommodation** (a two-berth booking issues two tickets), each carrying its own scannable QR, cached for offline render, validated at the edge (P-6, P-7). _Refs:_ belongs to a `Booking`; appears as one `ManifestEntry`; validated by a `TicketValidation`.

**Manifest / ManifestEntry** — the per-trip list of booked passengers, **derived from confirmed bookings** and pre-loaded to the edge before departure (O-5, P-6). _Refs:_ a `Trip`; entries reference `Ticket`/`Booking`. _(Ownership still open — stored projection vs rebuilt-on-demand; see decisions below.)_

**TicketValidation** — a record that a conductor scanned and validated a ticket; created **offline at the edge**, **idempotent**, and reconciled to the cloud on reconnect (O-8). _Refs:_ a `Ticket`; originates on the edge gateway. _(First consumer of the settled sync design — MVP Proposal §3.3.)_

## Payments (`payments` module — strongly consistent)

**Payment** — a charge against a booking through the PSP in **test mode** behind the abstraction; success is what confirms the booking (P-5, D-11). _Refs:_ a `Booking`; a PSP transaction reference. Single currency (EUR); split base/hospitality flow is cut.

## Identity (Keycloak realms — IdP-owned, D-05/D-06)

**Passenger** — a passenger-realm identity (email/password, Google, Apple); MFA optional (P-4). Owned by Keycloak; the monolith links bookings to it by **principal claim** (the Keycloak `sub`). _Refs:_ owns `Booking`s; has one `PassengerProfile`.

**PassengerProfile** — a thin **monolith-owned** row keyed by the principal claim, caching display attributes (name, contact) the manifest and booking views need without a round-trip to the IdP. **Not** the identity itself — Keycloak stays the system of record for passenger data, so GDPR export/deletion act there (D-05). _Refs:_ one per `Passenger` (by principal claim); read by `Manifest`.

**Operator** — an operator-realm identity with a minimal role (admin, conductor, cleaner); tokens **cached on the edge gateway** for full-trip offline auth (O-7). MFA/HR-provisioning deferred. _Refs:_ performs `TicketValidation`, `CleaningTask` completion, and admin trip actions.

> Realm isolation holds literally: passenger and operator tokens never cross, external tokens terminate at the edge, only **principal + realm claims** travel inward (D-05/D-06).

## Edge, IoT & offline (Train Edge Gateway, D-17)

**Device** — a physical sensor or actuator on the train: toilet-occupancy sensor, and the berth control panel (light, temperature, privacy lock) (P-8, P-9). _Refs:_ managed by the gateway through a `DeviceShadow`.

**DeviceShadow** — the reported (actual) + desired (target) state document per device; the gateway reconciles it, offline-native (P-9, P-10, D-17). _Refs:_ one per `Device`; desired state set by the passenger app.

**EdgeSyncRecord (local event log entry)** — the durable on-board log of edge-originated changes (validations, task completions, shadow writes) that reconcile upstream on reconnect (P-10, O-8). This is where the **HIGH RISK conflict-resolution / sync seam** lives (§7). _Refs:_ wraps `TicketValidation`, `CleaningTask`, `DeviceShadow` changes.

**CleaningTask** — a cleaning zone/task a cleaner marks complete on the device, offline, surfacing in the ops/admin view once reconciled (O-9). _Refs:_ a `Trip` (turnaround); completed by an `Operator`; carried by an `EdgeSyncRecord`.

> **Facility state** (toilet occupied/free, lounge occupancy) is deliberately **not an entity**: it is real-time only and **never persisted** — a passenger-privacy boundary that holds even in the Lab (P-8). It flows through a device shadow live and is never written to a store.

> **Cached ticket / booking** on the passenger device (P-7) is a **client-side projection** for offline render, not a server-owned entity; the source of truth stays in `orders/ticketing`.

## Cross-cutting (every module)

**OutboxEvent** — a domain event written to a module's outbox table **inside the same transaction** as the state change, then relayed to idempotent in-process consumers (D-02). The transport behind P-11's status updates, the manifest projection, and the offline reconciliations — and the accruing dataset the cut warehouse will later ingest. _Refs:_ emitted per module (`trips`, `inventory`, `orders/ticketing`, `payments`).

---

## Modelling decisions (this pass)

Resolved Jun 2026:

- **Accommodation is polymorphic** — one `InventoryUnit` with a type (seat / berth / suite-later), not separate `Seat`/`Berth` entities, so holds, seat-map, and booking stay generic.
- **One ticket per booked accommodation** — a two-berth booking issues two tickets, each with its own QR and manifest entry; per-passenger validation.
- **`Route` is a thin entity now** — `Trip` references it; a second route is just another row, no migration out of `Trip`.
- **A thin `PassengerProfile` is persisted** — monolith-owned, keyed by the principal claim, caching display attributes for the manifest; the identity itself stays in Keycloak.

### Still open

- **Manifest ownership:** stored projection materialised in `orders/ticketing` (one clean artifact for the edge to pre-load) vs rebuilt on demand from booking events. Left open this pass.
- **Edge vs cloud system of record after sync:** `TicketValidation` and `CleaningTask` are authored on the edge but reconcile into cloud owners — which side is authoritative post-sync, and how idempotency keys flow. **Deferred by decision** to the HIGH RISK conflict-resolution/sync seam (§7), to be settled on rungs 1–2 before the model commits.

## Explicitly excluded (boundary markers, MVP Proposal §4)

Loyalty (points, tiers, ledger) · hospitality & meals (orders, galley queue) · IM running information (delays, forecasts, paths request workflow) · Distribution/OSDM retail entities (channels, commission) · data-warehouse models · full fleet registry, maintenance, ERA authorisation / SSC gates · multi-currency · multi-tenant machinery beyond a `tenant/country` column · PNR · self-service changes/refunds. Each returns behind a seam the architecture already names.

---

_Catalogue only. Attributes, keys, and constraints follow once the entity set and the ownership boundaries are agreed. Anything here that contradicts a D-NN is a defect in this document, not a decision._
