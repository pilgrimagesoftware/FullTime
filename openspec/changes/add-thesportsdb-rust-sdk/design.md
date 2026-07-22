## Context

`Libs/openligadb/rust` (`openligadb.rs`) is the established pattern for a first-party Rust API client in this project: a standalone crate, submoduled under a `Libs/<provider>` meta-repo, with typed `serde` models under `src/models/`, a `thiserror`-based error type, and an optional `http-client` feature gating `reqwest`-backed `list`/`get` methods per model. The proposed `sportdb.rs` crate (`add-sportdb-dev-rust-sdk`) follows the same shape. TheSportsDB differs from both in one structural way: every endpoint is namespaced under an API key path segment (`/api/v1/json/{key}/...`), so the client needs an explicit key-configuration step that the other two SDKs don't require.

## Goals / Non-Goals

**Goals:**
- Ship a `thesportsdb.rs` crate with typed models for TheSportsDB's v1 resources: sports, leagues, league seasons, league tables/standings, teams, players, venues, season events, next events, previous events, event results, livescore, and search-by-name (teams, players, events, venues).
- Match the `openligadb.rs` crate's shape closely enough that a developer familiar with one can immediately work in the other: same `http-client` feature gate, same error-handling approach, same module layout, same scaffolding (git-flow branches, CI, changelog automation, community docs).
- Require an explicit API key at client construction (no hardcoded default), since TheSportsDB ties rate limits and endpoint availability to the key tier (free test key vs. paid).
- Keep model/serde types usable with zero networking dependencies (`default-features = false`).

**Non-Goals:**
- No caching, rate-limiting, or retry policy beyond what `reqwest` provides out of the box — consistent with the sibling crates.
- No abstraction shared between `openligadb.rs`, `sportdb.rs`, and `thesportsdb.rs` (e.g., a common client trait crate). Three independently-shaped provider APIs don't yet justify a shared abstraction; revisit only if a fourth provider reveals a genuine common pattern.
- No bundling or distribution of a real (paid) API key. The crate's tests and examples use the publicly documented free test key ("3") where TheSportsDB's docs permit it.
- No non-Rust clients (Swift/Go) in this change.

## Decisions

- **New `Libs/thesportsdb` meta-repo submodule, `rust` submodule inside it (`thesportsdb.rs`)** — mirrors `Libs/openligadb/rust` and the proposed `Libs/sportdb/rust`, keeping the same one-provider-per-meta-repo convention.
- **API key as a required constructor argument, not a default or env-var fallback inside the library** — the crate's client struct takes the key explicitly (e.g. `Client::new(api_key: impl Into<String>)`); callers decide whether to source it from an env var, config file, or the documented free test key. Alternative considered: reading from an environment variable inside the crate (rejected — implicit env reads inside a library are surprising and untestable; explicit construction keeps the crate a pure function of its inputs, consistent with "never expose internal errors" and least-surprise conventions).
- **`http-client` optional feature, default-on** — identical gating to `openligadb.rs` and the proposed `sportdb.rs`: `reqwest` + `url` behind the feature; model structs and their `serde` impls compile without it.
- **Per-resource modules under `src/models/`** (`sport.rs`, `league.rs`, `season.rs`, `table.rs`, `team.rs`, `player.rs`, `venue.rs`, `event.rs`, `livescore.rs`, `search.rs`), each exposing its own `list`/`get`/`search` network methods that accept the configured client/key — matches the sibling crates' per-model method placement.
- **`thesportsdb`'s numeric-string quirks handled at the model layer** — TheSportsDB's JSON responses represent many numeric fields (scores, IDs, counts) as strings. Models deserialize these through a shared `serde_with`-style helper in `src/util.rs` rather than requiring every model to hand-roll a custom deserializer, so the parsing quirk is isolated in one place.
- **`thiserror`-derived error enum in `src/error.rs`** — one variant per failure class (HTTP transport, deserialization, API error response, missing/invalid API key), matching the sibling crates. No panics, no `unwrap`/`expect` in library code.
- **Scaffold via `repo-setup-standard`** — git-flow, CI, `cargo-deny`/`cargo-audit`, git-cliff changelog/versioning, community docs — same as every other org Rust repo.

## Risks / Trade-offs

- [TheSportsDB's free tier has low rate limits and some endpoints are paid-key-only, so tests/examples built against the free test key may not exercise the full API surface] → Mitigate by documenting which endpoints require a paid key in the README and gating those integration tests behind an env var that skips them when no key is configured, rather than failing CI.
- [TheSportsDB's numeric-as-string JSON quirk is easy to miss per-model, causing silent parsing bugs] → Centralize the deserialization helper in `src/util.rs` (see Decisions) and cover it directly with unit tests, so every model reuses tested logic instead of reimplementing it.
- [Duplicating scaffolding effort across three provider crates instead of sharing tooling] → Accepted trade-off, consistent with the same call made for `sportdb.rs`: each SDK crate is independently versioned and released; shared scaffolding lives in the `repo-setup-standard` skill/template.

## Open Questions

- Does the project have (or need to acquire) a paid TheSportsDB API key for CI/integration testing, or should CI run exclusively against the free test key with paid-only endpoints skipped?
- Should the client expose a way to swap the API key at runtime (e.g., if a caller wants to upgrade from the free test key to a paid key without reconstructing the client), or is a new `Client` instance per key sufficient? Default to the latter unless a concrete caller need emerges.
