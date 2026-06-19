# Slice S-04 — Stand up the identity skeleton: Keycloak passenger + operator realms with edge token termination

> Status: Proposed
> Depends on: S-01
> Size: 1–2 days

## Outcome / why now
The realm-isolation boundary and token termination at the edge are real and tested from the first commit, so the no-cross-realm rule is never retrofitted onto live surfaces — one of the brutal-to-retrofit, not-negotiable items.
Serves: D-05 (isolated realms), D-06 (token termination + exchange), MVP §4 (not-negotiable).

## Situation
When the walking skeleton crosses realms — a passenger books, an operator validates — realm separation and external-token termination must be exercised once, minimally, before either surface carries real behaviour.

## Scope
**In:**
- Keycloak with a **passenger** realm (email/password + Google/Apple stub) and an **operator** realm (email + MFA), one user each, exported as reproducible config.
- Spring Security OAuth2 resource server terminating external tokens at the API edge.
- Token exchange (RFC 8693) to one internal issuer; only principal + realm **claim** travel inward.
- Cached-JWKS path so an operator token validates offline on the edge gateway.

**Out (deferred — and behind which seam):**
- eIDAS / government-ID schemes, HR provisioning, full MFA policy.
- Company realm + read-only DW boundary (DW is cut in the MVP).
- The full passenger-realm **offline-validation** story and token lifetimes (open in D-06 — ADR).

## Seams & contracts
- `dream-infra` repo: `keycloak/*-realm.json` (realm exports)
- Resource-server + token-exchange config in the `dream-backend` repo's edge module
- Cached-JWKS config on the Go edge gateway (`dream-edge` repo)

## Acceptance criteria
- Given a passenger-realm token, when it hits an operator-scoped endpoint, then it is rejected (no cross-realm acceptance).
- Given a valid passenger token at the API edge, when it is exchanged, then only principal + realm claims travel inward and the external token does not (termination).
- Given the operator realm's JWKS cached on the edge, when the gateway is offline, then a valid operator token still validates.
- Given the realm exports, when Keycloak is recreated from them, then both realms and users are reproduced identically.

## Definition of Done
- No-cross-realm acceptance proven by an automated test, not by convention.
- External tokens terminate at the edge; internal issuer is single (D-06).
- Realms reproducible from checked-in exports.

## Open decisions
- Passenger-realm offline-validation mechanism + token lifetimes — ADR needed (D-06 names this open; the edge offline-validation story constrains lifetimes).

## Converges into
The Keycloak realm exports, the resource-server/token-exchange config, the realm-isolation test.
