## ADDED Requirements

### Requirement: Model coverage matches the sportdb.dev resource surface
The crate SHALL provide typed, `serde`-derived model structs for every sportdb.dev resource: sports, countries, competitions, competition seasons, competition standings, competition fixtures, competition results, live events, live odds, match events, match lineups, match stats, team details, player details, and search results.

#### Scenario: Deserializing a competition standings response
- **WHEN** a caller deserializes a sportdb.dev competition standings API response into the crate's standings model
- **THEN** the resulting struct exposes every field present in the response without loss, using `Option<T>` for fields that are absent or nullable in observed responses

#### Scenario: Deserializing a live odds response
- **WHEN** a caller deserializes a sportdb.dev live odds API response into the crate's live odds model
- **THEN** the resulting struct exposes every field present in the response without loss, using `Option<T>` for fields that are absent or nullable in observed responses

### Requirement: Models are usable without the networking feature
The crate SHALL compile and expose all model structs and their `serde` implementations when built with `default-features = false`, with zero networking dependencies in that configuration.

#### Scenario: Building without the http-client feature
- **WHEN** a consumer depends on the crate with `default-features = false`
- **THEN** the crate builds successfully, model types are available, and no `reqwest` or `url` dependency is pulled in

### Requirement: Network methods available under the http-client feature
The crate SHALL provide `list` and/or `get` style async network methods per resource model, gated behind the default-on `http-client` feature, matching the pattern used by the existing `openligadb.rs` crate.

#### Scenario: Listing competitions with the default feature set
- **WHEN** a consumer depends on the crate with default features enabled and calls the competitions `list` method
- **THEN** the method issues an HTTP request to the sportdb.dev API and returns a `Result` containing the deserialized list of competitions or a crate error

#### Scenario: Fetching a single match's details
- **WHEN** a consumer calls the match details `get` method with a valid match identifier
- **THEN** the method issues an HTTP request to the sportdb.dev API and returns a `Result` containing the deserialized match details or a crate error

### Requirement: Errors are typed and non-panicking
The crate SHALL report all failure conditions (HTTP transport failure, response deserialization failure, API-level error response) through a `thiserror`-derived error enum returned in a `Result`, and SHALL NOT panic, `unwrap`, or `expect` on any externally-sourced input.

#### Scenario: Upstream API returns a non-success status
- **WHEN** the sportdb.dev API responds with a non-success HTTP status to a network method call
- **THEN** the method returns an `Err` variant identifying the failure class instead of panicking or silently returning a default value

#### Scenario: Response body fails to deserialize
- **WHEN** the sportdb.dev API returns a response body that does not match the expected model shape
- **THEN** the method returns an `Err` variant identifying the deserialization failure instead of panicking
