## Context

`Libs/openligadb/rust` (`openligadb.rs`) is the established pattern for a first-party Rust API client in this project: a standalone crate, submoduled under a `Libs/<provider>` meta-repo, with typed `serde` models under `src/models/`, a `thiserror`-based error type, and an optional `http-client` feature gating `reqwest`-backed `list`/`get` methods per model. sportdb.dev is a broader multi-sport data feed (sports, countries, competitions/seasons, standings, fixtures, results, live scores/odds, match events/lineups/stats, team/player details, search) with no first-party Rust client today. The existing `sportdbdotdev` MCP server (Flashscore-backed tools) is a working reference for the API's shape but is not itself a dependency of the new crate — the crate talks to the sportdb.dev HTTP API directly.

## Goals / Non-Goals

**Goals:**
- Ship a `sportdb.rs` crate with typed models for every sportdb.dev resource covering: sports, countries, competitions, competition seasons, competition standings, competition fixtures, competition results, live events, live odds, match events, match lineups, match stats, team details, player details, and search.
- Match the `openligadb.rs` crate's shape closely enough that a developer familiar with one can immediately work in the other: same `http-client` feature gate, same error-handling approach, same module layout, same scaffolding (git-flow branches, CI, changelog automation, community docs).
- Keep model/serde types usable with zero networking dependencies (`default-features = false`), so the crate cross-compiles to targets `reqwest` can't reach.

**Non-Goals:**
- No caching, rate-limiting, or retry policy beyond what `reqwest` provides out of the box — that's a caller concern, consistent with `openligadb.rs`.
- No abstraction shared between `openligadb.rs` and `sportdb.rs` (e.g., a common trait crate). The two APIs model different domains; premature sharing would be speculative given only two data points.
- No changes to the `sportdbdotdev` MCP server or its tool surface — it is a reference for API shape only, not a dependency.
- No non-Rust clients (Swift/Go) in this change; those would be separate follow-up changes if/when needed, per the `add-openligadb-swift-go-clients` precedent.

## Decisions

- **New `Libs/sportdb` meta-repo submodule, `rust` submodule inside it (`sportdb.rs`)** — mirrors `Libs/openligadb/rust`. Keeps room for future non-Rust clients under the same meta-repo without restructuring, same as `openligadb`.
- **`http-client` optional feature, default-on** — identical gating to `openligadb.rs`: `reqwest` + `url` behind the feature; model structs and their `serde` impls compile without it. Alternative considered: always-on networking (simpler `Cargo.toml`, rejected — breaks WASM/no-network use cases the sibling crate already supports and that consumers may expect for symmetry).
- **Per-resource modules under `src/models/`** (`sport.rs`, `country.rs`, `competition.rs`, `season.rs`, `standing.rs`, `fixture.rs`, `result.rs`, `live_event.rs`, `live_odds.rs`, `match_event.rs`, `match_lineup.rs`, `match_stats.rs`, `team.rs`, `player.rs`, `search.rs`), each exposing its own `list`/`get` network methods — matches `openligadb.rs`'s per-model method placement rather than a single monolithic client struct. Alternative considered: one `Client` struct with methods per resource (more conventional for some Rust SDKs, rejected for consistency with the sibling crate and because it avoids a shared-state struct that must be threaded through every call site).
- **`thiserror`-derived error enum in `src/error.rs`** — one variant per failure class (HTTP transport, deserialization, API error response), matching `openligadb.rs`. No panics, no `unwrap`/`expect` in library code, per the org's Rust rules.
- **Scaffold via `repo-setup-standard`** — git-flow (`master`/`develop`), CI (fmt/clippy/test), `cargo-deny`/`cargo-audit`, git-cliff-driven changelog and versioning, README/CONTRIBUTING/CODE_OF_CONDUCT/SECURITY/LICENSE — same as every other org Rust repo, so it needs no bespoke tooling decisions.

## Risks / Trade-offs

- [sportdb.dev's API surface may be undocumented or unstable in places (it backs live scores/odds), risking model drift after this crate ships] → Mitigate by deriving models directly from observed API responses (via the existing MCP server's tool outputs as a reference) and covering serde round-trips with tests per model; treat breaking upstream changes as normal semver-major bumps via git-cliff, same as any other dependency drift.
- [Live-data resources (live events, live odds) may have looser or more volatile schemas than static resources (sports, countries)] → Model optional/nullable fields conservatively (`Option<T>`) rather than assuming completeness, and add fixture-based tests using captured real responses.
- [Duplicating scaffolding effort across `openligadb.rs` and `sportdb.rs` instead of sharing tooling] → Accepted trade-off: each SDK crate is independently versioned and released; shared scaffolding lives in the `repo-setup-standard` skill/template, not in a shared crate dependency.

## Open Questions

- Does sportdb.dev require an API key or auth token for any of these endpoints? If so, the client needs a credential-configuration mechanism (constructor param or env var) — to be confirmed against the live API before implementation and reflected in tasks.md.
- Are there rate limits on sportdb.dev that the crate should surface as a distinct error variant (vs. a generic HTTP error), so callers can implement backoff themselves?
