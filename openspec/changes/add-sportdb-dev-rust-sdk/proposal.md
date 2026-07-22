## Why

`Libs` already ships a first-party Rust client for OpenLigaDB (`Libs/openligadb/rust`, `openligadb.rs`), giving the Rust app and SDK consumers typed models and network methods instead of hand-rolled HTTP calls. [sportdb.dev](https://sportdb.dev) exposes a broader, multi-sport data feed (Flashscore-backed: competitions, fixtures, results, standings, live scores/odds, match events/lineups/stats, player and team details, search) that the project currently has no first-party client for. Adding a Rust SDK for sportdb.dev, following the same pattern as `openligadb.rs`, lets Rust consumers (the desktop app, other Libs crates) pull sportdb.dev data through typed models and `list`/`get` style methods instead of duplicating parsing and transport code per caller.

## What Changes

- Add a new `Libs/sportdb` meta-repo submodule containing a `rust` submodule (`sportdb.rs`) that mirrors the `openligadb.rs` crate layout: `src/models/*` for typed data structures with `serde` derives, `src/error.rs`, `src/constants.rs`, `src/util.rs`, and per-model `list`/`get` network methods.
- Model coverage matches the sportdb.dev API surface: sports, countries, competitions (and their seasons), competition standings, fixtures, results, live events, live odds, match details (events, lineups, stats), team details, player details, and search.
- Follow the existing `http-client` optional-feature pattern: model/serde types compile and are usable with `default-features = false` (no networking dependency), while the default feature set adds `reqwest`-backed network methods per model.
- Scaffold `sportdb.rs` as its own repository per the org's Rust conventions (`repo-setup-standard`, `openspec-propose` patterns already used for `openligadb.rs`): `develop`/`master` git-flow branches, CI (fmt/clippy/test), `cargo-deny`/`cargo-audit`, git-cliff changelog automation, README, CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, LICENSE.
- Update the top-level `Libs` meta-repo (`.gitmodules`, README) to register and describe the new `sportdb` submodule, and update this top-level `FullTime` repo's submodule pointer for `Libs` once `sportdb` lands.

## Capabilities

### New Capabilities
- `sportdb-client-core`: defines the model surface (sports, countries, competitions, seasons, standings, fixtures, results, live scores, live odds, match events/lineups/stats, team details, player details, search) and `list`/`get` client behavior that the `sportdb.rs` crate must provide, and the `http-client` feature gating pattern (models usable without networking; network methods opt-in via default feature).

### Modified Capabilities
- (none — no existing top-level specs predate this change)

## Impact

- **Libs**: new `sportdb` submodule (meta-repo) with a `rust` submodule inside it; `.gitmodules` and top-level README updated to reference it, matching how `openligadb` is registered today.
- **New repository**: `sportdb.rs`, scaffolded with git-flow branches, CI, and release tooling matching `openligadb.rs`.
- **Apps/rust**: unblocks consuming sportdb.dev via a first-party Rust client instead of ad hoc networking code (no app-side changes required by this change itself).
- No changes to the existing `openligadb.rs` client's public API; this is a new, independent crate.
