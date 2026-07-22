## Why

`Libs` already ships first-party Rust clients for OpenLigaDB (`Libs/openligadb/rust`, `openligadb.rs`) and, pending implementation, sportdb.dev (`Libs/sportdb/rust`, see `add-sportdb-dev-rust-sdk`). [TheSportsDB](https://www.thesportsdb.com) is a distinct, widely used multi-sport reference database (leagues, teams, players, venues, season events/results, livescore, and search-by-name lookups, keyed by an API key) that the project has no first-party client for. Adding a Rust SDK for TheSportsDB, following the same crate pattern as the existing SDKs, lets Rust consumers pull TheSportsDB's reference and event data through typed models and `list`/`get` style methods instead of duplicating parsing and transport code per caller.

## What Changes

- Add a new `Libs/thesportsdb` meta-repo submodule containing a `rust` submodule (`thesportsdb.rs`) that mirrors the `openligadb.rs` crate layout: `src/models/*` for typed data structures with `serde` derives, `src/error.rs`, `src/constants.rs`, `src/util.rs`, and per-model `list`/`get`/`search` network methods.
- Model coverage matches TheSportsDB's v1 API surface: sports, leagues (and league seasons/tables), teams, players, venues, season events, next/previous events, event results, livescore, and search-by-name endpoints (teams, players, events, venues).
- The client carries a required API key (TheSportsDB gates all endpoints behind a key path segment, with a shared free test key for limited endpoints and paid keys for full access) — the crate's constructor/config takes the key rather than hardcoding one.
- Follow the existing `http-client` optional-feature pattern: model/serde types compile and are usable with `default-features = false` (no networking dependency), while the default feature set adds `reqwest`-backed network methods per model.
- Scaffold `thesportsdb.rs` as its own repository per the org's Rust conventions (`repo-setup-standard`, `openspec-propose` patterns already used for `openligadb.rs`): `develop`/`master` git-flow branches, CI (fmt/clippy/test), `cargo-deny`/`cargo-audit`, git-cliff changelog automation, README, CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, LICENSE.
- Update the top-level `Libs` meta-repo (`.gitmodules`, README) to register and describe the new `thesportsdb` submodule, and update this top-level `FullTime` repo's submodule pointer for `Libs` once `thesportsdb` lands.

## Capabilities

### New Capabilities
- `thesportsdb-client-core`: defines the model surface (sports, leagues, league seasons/tables, teams, players, venues, season events, next/previous events, results, livescore, search-by-name) and `list`/`get`/`search` client behavior that the `thesportsdb.rs` crate must provide, the API-key configuration requirement, and the `http-client` feature gating pattern (models usable without networking; network methods opt-in via default feature).

### Modified Capabilities
- (none — no existing top-level specs predate this change)

## Impact

- **Libs**: new `thesportsdb` submodule (meta-repo) with a `rust` submodule inside it; `.gitmodules` and top-level README updated to reference it, matching how `openligadb` is registered today.
- **New repository**: `thesportsdb.rs`, scaffolded with git-flow branches, CI, and release tooling matching `openligadb.rs`.
- **Apps/rust**: unblocks consuming TheSportsDB via a first-party Rust client instead of ad hoc networking code (no app-side changes required by this change itself).
- No changes to the existing `openligadb.rs` client or the proposed `sportdb.rs` client; this is a new, independent crate with its own API key configuration.
