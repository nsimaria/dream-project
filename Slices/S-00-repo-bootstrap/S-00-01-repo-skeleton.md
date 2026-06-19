# Slice S-00-01 — Repo set, `dream-contracts` repo, dream-orchestration skeleton & submodule wiring

> Status: Proposed
> Depends on: —
> Size: ≤ ½ day

## Outcome / why now
The seven repositories exist on trunk, the `dream-contracts` repo is the designated source of truth wired into each consumer as a pinned submodule, and the dream-orchestration repo has a place for `make train` — so every later subslice has a known repo home for its stack and a working contract-sharing mechanism.
Serves: MVP §6 (M0), MVP Technical Choices (the four-language map; multi-repo + submodule + dream-orchestration).

## Situation
Nothing exists yet. The repositories, the contract-sharing mechanism (submodule), and the dream-orchestration entry point must exist before any build wiring or stack project can be added — otherwise stacks have nowhere to live and contracts have no shared home.

## Scope
**In:**
- **Create seven repos on trunk:** `dream-backend`, `dream-passenger-web-app`, `dream-operator-android-app`, `dream-edge`, `dream-contracts`, `dream-infra`, `dream-orchestration`. Each with a root `.gitignore` and `README.md`.
- **`dream-contracts` repo placeholder layout:** `openapi/`, `asyncapi/`, `shadow/` directories with placeholder content (specs themselves land in S-02).
- **Submodule wiring:** add `dream-contracts` as a **git submodule pinned at an explicit SHA** in each consuming repo (`dream-backend`, `dream-passenger-web-app`, `dream-operator-android-app`, `dream-edge`, `dream-orchestration`), recorded in each `.gitmodules`.
- **dream-orchestration skeleton:** a `Makefile` placeholder and a sibling-checkout script that knows the repo set and their pinned SHAs (targets filled in S-00-06).

**Out (deferred — and behind which seam):**
- Any per-stack build wiring or project → **S-00-02…05**.
- Contract spec content + codegen → **S-02**.
- The working `make verify` / `make train` targets → **S-00-06**.
- `dream-infra` Terraform skeleton → **S-01**.

## Seams & contracts
- Seven repo roots, each `/.gitignore`, `/README.md`
- `contracts/{openapi,asyncapi,shadow}/` placeholder layout
- `.gitmodules` in each consuming repo (pinned `dream-contracts` SHA)
- `dream-orchestration/Makefile` placeholder + sibling-checkout script + pinned-SHA manifest

## Acceptance criteria
- Given clean clones, when the repo set is inspected, then all seven repos exist on trunk with the agreed roots.
- Given a consuming repo cloned with `--recurse-submodules`, when inspected, then the `dream-contracts` submodule resolves to its pinned SHA.
- Given the dream-orchestration repo, when its sibling-checkout script runs, then it resolves each sibling repo at the pinned SHA (build/test targets not yet required).

## Definition of Done
- On trunk in every repo; layout matches the structure S-00 names.
- `dream-contracts` wired as a pinned submodule in every consumer; resolution is deterministic.

## Open decisions
- Repo host / group structure — one GitLab group, seven projects assumed (MVP Technical Choices: GitLab CI).
- Where the pinned-SHA manifest for siblings lives (`dream-orchestration` `Makefile` vars vs a separate file) — recorded in the dream-orchestration repo.

## Converges into
The seven repositories and their root dotfiles, the `dream-contracts` placeholder layout, the pinned `dream-contracts` submodule wiring in each consumer, and the dream-orchestration sibling-checkout skeleton.
