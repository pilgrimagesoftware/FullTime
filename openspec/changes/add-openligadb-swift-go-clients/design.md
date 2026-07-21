## Context

`Libs/openligadb` is a meta-repo (like `dtrpg-sdk`) whose `rust` submodule (`openligadb.rs`) is the reference client for the [OpenLigaDB API](https://api.openligadb.de/index.html). Its models (`League`, `Sport`, `Team`, `Match`, `Group`, `Goal`, `Result`, `Table`, `Location`) are plain serde structs with `Option<T>` fields for anything OpenLigaDB can omit, `i32`/`String` id and name fields, and network methods (`list`, `get`, etc.) gated behind an `http-client` feature so the model layer stays usable without pulling in an HTTP stack. This change adds Swift and Go clients that reproduce that same model set and network surface as new sibling submodules, following the `dtrpg-sdk/rust`-as-reference pattern already established in `docs/openspec.md`.

## Goals / Non-Goals

**Goals:**
- Swift and Go clients expose the same OpenLigaDB entities (leagues, sports, teams, matches, groups, goals, results, tables, locations) with the same optionality as the Rust client.
- Each new client separates model/decoding types from the HTTP transport, so consumers can use the models without a network dependency where the language ecosystem makes that practical.
- Each new repository is scaffolded with the org's standard git-flow branches, CI, changelog automation (`git-cliff`-equivalent or language-idiomatic tooling), README, CONTRIBUTING, and LICENSE, matching `openligadb.rs`.
- `Libs/openligadb` registers both as submodules alongside `rust`.

**Non-Goals:**
- No changes to the OpenLigaDB API surface itself or to the Rust client's behavior.
- No app-side integration in `Apps/swift` or elsewhere in this change — that is follow-up work once the client exists.
- No requirement that Swift/Go match Rust's internal implementation (feature-gating mechanism, module layout) — only its observable model and client behavior.

## Decisions

- **One shared capability, per-language repos**: model the contract as a single `openligadb-client-parity` capability rather than one spec per language, since the requirement is behavioral equivalence across clients, not language-specific behavior. Each language's repo implements against the same spec.
  - Alternative considered: separate `openligadb-swift-client` / `openligadb-go-client` specs. Rejected — it would duplicate near-identical requirements and let the two implementations drift without a shared contract to check both against.
- **Field naming**: Swift and Go clients use each language's idiomatic case (`camelCase` for Swift properties via `Codable` key mapping, exported `PascalCase` struct fields with `json` tags for Go) while preserving the same optionality and semantics as the Rust `snake_case`-avoiding, OpenLigaDB-camelCase-mapped fields.
- **Networking separation**: Swift uses `URLSession` behind a protocol/thin wrapper so tests and non-networked consumers can substitute a fake transport; Go exposes model decoding independent of its default `net/http`-based client via an injectable `HTTPDoer`-style interface. Both mirror the Rust crate's `http-client` feature intent (models usable standalone) without requiring literal feature flags, since Swift Package Manager and Go modules don't have Cargo-style optional features.
- **Repo scaffolding**: reuse the `repo-setup-standard` skill/pattern already applied to `openligadb.rs` (git-flow `master`/`develop`, CI, `AGENTS.md`/`CLAUDE.md`, CONTRIBUTING/SECURITY/CODE_OF_CONDUCT, release automation) for both new repos, adapted to Swift Package Manager and Go modules conventions respectively.

## Risks / Trade-offs

- [Model drift between Rust, Swift, and Go as OpenLigaDB's API evolves] → the `openligadb-client-parity` spec is the shared source of truth; changes to it require updating all three clients, and future OpenLigaDB API changes should land as spec deltas reviewed against all three.
- [Swift/Go ecosystems lack a direct equivalent to Cargo's optional-feature gating] → accept an idiomatic per-language substitute (protocol/interface injection) rather than forcing a literal feature-flag mechanism; document the substitution in each repo's README.
- [Two new repositories double the release/CI surface to maintain] → scaffold both from the same `repo-setup-standard` pattern so maintenance stays mechanical rather than bespoke per repo.

## Migration Plan

1. Create `openligadb.swift` and `openligadb.go` repositories, scaffolded per `repo-setup-standard`.
2. Implement models and network methods against the `openligadb-client-parity` spec, using `openligadb.rs` as the behavioral reference.
3. Register both as submodules under `Libs/openligadb`, update `.gitmodules` and the meta-repo README.
4. No rollback complexity beyond removing the submodule references — the Rust client and existing app behavior are untouched.

## Open Questions

- Final published package names for the Swift (SwiftPM) and Go (module path) clients — deferred to repo creation time, following the `openligadb.rs` / `openligadb-rs` naming precedent.
