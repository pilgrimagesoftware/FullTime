## 1. Repository scaffolding

- [ ] 1.1 Create the `sportdb.rs` GitHub repository under the pilgrimagesoftware org
- [ ] 1.2 Run `repo-setup-standard` to scaffold git-flow branches (`master`/`develop`), CI workflows (fmt/clippy/test), dependabot, `cargo-deny`/`cargo-audit`, git-cliff changelog automation, README, CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, LICENSE, and AGENTS.md/CLAUDE.md
- [ ] 1.3 Add `Cargo.toml` with crate metadata (name `sportdb`, description, keywords, categories, authors, license, repository/documentation/homepage URLs) mirroring `openligadb.rs`
- [ ] 1.4 Add the `http-client` optional feature (default-on) gating `reqwest` and `url`, plus base dependencies (`chrono`, `serde`, `serde_json`, `thiserror`)
- [ ] 1.5 Confirm sportdb.dev's authentication requirements (API key/token) and rate-limit behavior; update design.md's Open Questions with the findings before proceeding

## 2. Core error and utility modules

- [ ] 2.1 Implement `src/error.rs` with a `thiserror`-derived error enum covering transport failure, deserialization failure, and API-level error responses
- [ ] 2.2 Implement `src/util.rs` for shared request/response helpers (base URL, auth header injection if required, JSON decoding)
- [ ] 2.3 Implement `src/constants.rs` for the sportdb.dev base URL and any fixed path segments

## 3. Static resource models

- [ ] 3.1 Implement `src/models/sport.rs` (list sports) with `serde` model and `list` network method
- [ ] 3.2 Implement `src/models/country.rs` (list countries) with `serde` model and `list` network method
- [ ] 3.3 Implement `src/models/competition.rs` (list competitions, get competition) with `serde` model and network methods
- [ ] 3.4 Implement `src/models/season.rs` (list competition seasons) with `serde` model and network method

## 4. Fixture, result, and standings models

- [ ] 4.1 Implement `src/models/fixture.rs` (competition fixtures) with `serde` model and network method
- [ ] 4.2 Implement `src/models/result.rs` (competition results) with `serde` model and network method
- [ ] 4.3 Implement `src/models/standing.rs` (competition standings) with `serde` model and network method

## 5. Live data models

- [ ] 5.1 Implement `src/models/live_event.rs` (live scores/events) with `serde` model and network method
- [ ] 5.2 Implement `src/models/live_odds.rs` (live odds) with `serde` model and network method

## 6. Match detail models

- [ ] 6.1 Implement `src/models/match_event.rs` (match events) with `serde` model and network method
- [ ] 6.2 Implement `src/models/match_lineup.rs` (match lineups) with `serde` model and network method
- [ ] 6.3 Implement `src/models/match_stats.rs` (match stats) with `serde` model and network method

## 7. Team, player, and search models

- [ ] 7.1 Implement `src/models/team.rs` (team details) with `serde` model and network method
- [ ] 7.2 Implement `src/models/player.rs` (player details) with `serde` model and network method
- [ ] 7.3 Implement `src/models/search.rs` (search) with `serde` model and network method

## 8. Testing

- [ ] 8.1 Add `serde` round-trip tests per model using captured real sportdb.dev responses as fixtures
- [ ] 8.2 Add tests confirming the crate builds and model types are usable with `default-features = false`
- [ ] 8.3 Add integration tests (gated behind `http-client`) covering error paths: non-success HTTP status and malformed response body
- [ ] 8.4 Run `cargo test --all-features --workspace` and `cargo test --no-default-features` in CI

## 9. Documentation and release

- [ ] 9.1 Write README usage examples for at least one static resource (e.g. competitions) and one live resource (e.g. live odds)
- [ ] 9.2 Document the `http-client` feature gate in README, matching `openligadb.rs`'s explanation
- [ ] 9.3 Cut an initial `0.0.x` release via the `prepare-release` workflow and tag per `docs/git-flow.md`

## 10. Meta-repo integration

- [ ] 10.1 Create the `Libs/sportdb` meta-repo submodule with a `.gitmodules` referencing `rust` (`sportdb.rs`)
- [ ] 10.2 Add `rust` as a submodule of `Libs/sportdb`
- [ ] 10.3 Update `Libs` top-level `.gitmodules` and README to register `sportdb` alongside `openligadb`
- [ ] 10.4 Update this repository's `Libs` submodule pointer once `Libs/sportdb` is committed and pushed
