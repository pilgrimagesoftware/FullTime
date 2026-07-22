## ADDED Requirements

### Requirement: Model coverage matches the TheSportsDB v1 resource surface
The crate SHALL provide typed, `serde`-derived model structs for every TheSportsDB v1 resource: sports, leagues, league seasons, league tables/standings, teams, players, venues, season events, next events, previous events, event results, livescore, and search-by-name results (teams, players, events, venues).

#### Scenario: Deserializing a league table response
- **WHEN** a caller deserializes a TheSportsDB league table API response into the crate's table model
- **THEN** the resulting struct exposes every field present in the response without loss, including numeric fields represented as JSON strings, using `Option<T>` for fields that are absent or nullable in observed responses

#### Scenario: Deserializing a livescore response
- **WHEN** a caller deserializes a TheSportsDB livescore API response into the crate's livescore model
- **THEN** the resulting struct exposes every field present in the response without loss, including numeric fields represented as JSON strings

### Requirement: Client requires an explicit API key
The crate SHALL require an API key to be supplied when constructing a client, and SHALL NOT embed, default to, or silently read an API key from the environment inside the library.

#### Scenario: Constructing a client with an API key
- **WHEN** a consumer constructs the crate's client with an API key value
- **THEN** all subsequent network methods invoked on that client include the configured key in the request path

#### Scenario: No API key provided
- **WHEN** a consumer attempts to construct the crate's client without supplying an API key
- **THEN** construction fails to compile or returns an error, rather than falling back to an implicit default

### Requirement: Models are usable without the networking feature
The crate SHALL compile and expose all model structs and their `serde` implementations when built with `default-features = false`, with zero networking dependencies in that configuration.

#### Scenario: Building without the http-client feature
- **WHEN** a consumer depends on the crate with `default-features = false`
- **THEN** the crate builds successfully, model types are available, and no `reqwest` or `url` dependency is pulled in

### Requirement: Network methods available under the http-client feature
The crate SHALL provide `list`, `get`, and/or `search` style async network methods per resource model, gated behind the default-on `http-client` feature, matching the pattern used by the existing `openligadb.rs` crate.

#### Scenario: Listing a league's season events with the default feature set
- **WHEN** a consumer depends on the crate with default features enabled and calls the season events `list` method for a given league and season
- **THEN** the method issues an HTTP request to TheSportsDB API and returns a `Result` containing the deserialized list of events or a crate error

#### Scenario: Searching for a team by name
- **WHEN** a consumer calls the team `search` method with a team name
- **THEN** the method issues an HTTP request to TheSportsDB API and returns a `Result` containing the deserialized matching teams or a crate error

### Requirement: Errors are typed and non-panicking
The crate SHALL report all failure conditions (HTTP transport failure, response deserialization failure, API-level error response, invalid or rate-limited API key) through a `thiserror`-derived error enum returned in a `Result`, and SHALL NOT panic, `unwrap`, or `expect` on any externally-sourced input.

#### Scenario: Upstream API returns a non-success status
- **WHEN** TheSportsDB API responds with a non-success HTTP status to a network method call
- **THEN** the method returns an `Err` variant identifying the failure class instead of panicking or silently returning a default value

#### Scenario: Response body fails to deserialize
- **WHEN** TheSportsDB API returns a response body that does not match the expected model shape
- **THEN** the method returns an `Err` variant identifying the deserialization failure instead of panicking
