## 1. Repository scaffolding

- [ ] 1.1 Create the `thesportsdb.rs` GitHub repository under the pilgrimagesoftware org
- [ ] 1.2 Run `repo-setup-standard` to scaffold git-flow branches (`master`/`develop`), CI workflows (fmt/clippy/test), dependabot, `cargo-deny`/`cargo-audit`, git-cliff changelog automation, README, CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, LICENSE, and AGENTS.md/CLAUDE.md
- [ ] 1.3 Add `Cargo.toml` with crate metadata (name `thesportsdb`, description, keywords, categories, authors, license, repository/documentation/homepage URLs) mirroring `openligadb.rs`
- [ ] 1.4 Add the `http-client` optional feature (default-on) gating `reqwest` and `url`, plus base dependencies (`chrono`, `serde`, `serde_json`, `thiserror`)
- [ ] 1.5 Confirm TheSportsDB's free-vs-paid key tiering and rate limits; document which endpoints require a paid key and update design.md's Open Questions with the findings before proceeding

## 2. Core client, error, and utility modules

- [ ] 2.1 Implement `src/client.rs` (or equivalent) holding the required API key and base request logic used by every model's network methods
- [ ] 2.2 Implement `src/error.rs` with a `thiserror`-derived error enum covering transport failure, deserialization failure, API-level error responses, and invalid/rate-limited API key
- [ ] 2.3 Implement `src/util.rs` including a shared deserialization helper for TheSportsDB's numeric-as-string JSON fields
- [ ] 2.4 Implement `src/constants.rs` for TheSportsDB's base URL and fixed path segments

## 3. Reference resource models

- [ ] 3.1 Implement `src/models/sport.rs` (list sports) with `serde` model and `list` network method
- [ ] 3.2 Implement `src/models/league.rs` (list leagues, get league) with `serde` model and network methods
- [ ] 3.3 Implement `src/models/season.rs` (list league seasons) with `serde` model and network method
- [ ] 3.4 Implement `src/models/table.rs` (league table/standings) with `serde` model and network method

## 4. Team, player, and venue models

- [ ] 4.1 Implement `src/models/team.rs` (get team, search teams) with `serde` model and network methods
- [ ] 4.2 Implement `src/models/player.rs` (get player, search players) with `serde` model and network methods
- [ ] 4.3 Implement `src/models/venue.rs` (get venue) with `serde` model and network method

## 5. Event and livescore models

- [ ] 5.1 Implement `src/models/event.rs` covering season events, next events, previous events, event results, and event search, each with `serde` model and network methods
- [ ] 5.2 Implement `src/models/livescore.rs` (livescore) with `serde` model and network method

## 6. Testing

- [ ] 6.1 Add unit tests for the numeric-as-string deserialization helper in `src/util.rs`
- [ ] 6.2 Add `serde` round-trip tests per model using captured real TheSportsDB responses as fixtures
- [ ] 6.3 Add tests confirming the crate builds and model types are usable with `default-features = false`
- [ ] 6.4 Add integration tests (gated behind `http-client`) covering error paths: non-success HTTP status, malformed response body, and invalid API key; skip paid-key-only endpoint tests when no paid key is configured
- [ ] 6.5 Run `cargo test --all-features --workspace` and `cargo test --no-default-features` in CI

## 7. Documentation and release

- [ ] 7.1 Write README usage examples for at least one reference resource (e.g. leagues) and one event resource (e.g. season events), including how to supply an API key
- [ ] 7.2 Document the `http-client` feature gate and the free-vs-paid key tiers in README
- [ ] 7.3 Cut an initial `0.0.x` release via the `prepare-release` workflow and tag per `docs/git-flow.md`

## 8. Meta-repo integration

- [ ] 8.1 Create the `Libs/thesportsdb` meta-repo submodule with a `.gitmodules` referencing `rust` (`thesportsdb.rs`)
- [ ] 8.2 Add `rust` as a submodule of `Libs/thesportsdb`
- [ ] 8.3 Update `Libs` top-level `.gitmodules` and README to register `thesportsdb` alongside `openligadb` (and `sportdb`, if already landed)
- [ ] 8.4 Update this repository's `Libs` submodule pointer once `Libs/thesportsdb` is committed and pushed
