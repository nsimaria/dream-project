# Slices

_Draft · June 2026 · Companion to MVP Proposal.md, User Stories.md, MVP Technical Choices.md_

A **slice spec** is a one-page, fixed-template artifact for a single unit of foundational/enabler work — the kind that has no honest user persona. It sits *above* the code as temporary scaffolding and is retired once the code, contract, and test it produced exist.

## Why this exists (and what it is not)

Three artifact types, three jobs:

- **ADR** (the `D-NN` records in Target Architecture.md) records a **decision**.
- **Contract** (OpenAPI / AsyncAPI / JSON Schema) records an **interface**.
- **Slice spec** (here) records a **unit of work** — its boundary, its acceptance, the seams it touches.

A slice spec is **not** a user story. We use the story format only where a real persona consumes the deliverable (passenger, ops admin, conductor — see User Stories.md — and developer-experience tooling, which has the developer as its genuine customer). Everything else foundational is a slice.

## The convergence rule (non-negotiable)

Code is the primary documentation. So every field in a slice spec either:

1. points at an **executable artifact** that becomes the durable spec (a contract file, a test, an ArchUnit rule, a CI config), or
2. is **ephemeral thinking** deleted once the code lands.

Acceptance criteria are written as Given/When/Then so they can be lifted verbatim into the test; once there, the prose copy carries no information the test doesn't, and the slice is retired. A slice spec that becomes a maintained document drifting from the code has failed.

Put depth where it's executable — the contract and the acceptance scenarios — and keep the prose thin. Detail in a spec file or a test can't lie; detail as narrative rots.

## Sizing

A slice is "small enough" if it can be merged to trunk behind a flag within roughly a day or two and demoed against the rung-1 sim. If not, split it.

## M0 foundation set (this folder)

The walking skeleton (book → pay → ticket → validate offline at the edge → advance state → passenger sees it) consumes all four of these. Build them first.

| ID | Slice | Depends on |
|---|---|---|
| S-00 | Repo bootstrap — stand up each per-surface repo one at a time (hello-world app + own CI), then wrap them into dream-orchestration; every stack boots and proves itself alive | — |
| S-01 | Module boundaries + CI gates + infra skeleton | S-00 |
| S-02 | Contract-first toolchain + conformance harness skeleton | S-01 |
| S-03 | Rung-1 in-memory train sim + `make train` | S-02 |
| S-04 | Identity skeleton — Keycloak passenger + operator realms | S-01 |

Use `_TEMPLATE.md` for new slices.
