## ADDED Requirements

### Requirement: Shared model surface
Every OpenLigaDB client library (Rust, Swift, Go) SHALL expose the same set of model types — League, Sport, Team, Group, Match, Goal, Result, Table, Location — with equivalent fields and optionality to the reference Rust client (`openligadb.rs`).

#### Scenario: League model field parity
- **WHEN** the Swift or Go client decodes a league returned by OpenLigaDB's `getavailableleagues` endpoint
- **THEN** the resulting model exposes an id, an optional name, an optional shortcut, an optional season, and a required sport, matching the field set and optionality of the Rust client's `League` struct

#### Scenario: Optional fields decode as absent, not error
- **WHEN** an OpenLigaDB response omits a field that the Rust client models as `Option<T>`
- **THEN** the Swift and Go clients decode the corresponding field as absent (`nil`/zero value) rather than failing to decode

### Requirement: Model layer usable without network transport
Each client SHALL make model decoding usable independent of any specific HTTP transport, so consumers can decode OpenLigaDB JSON through their own transport without requiring the client's default network stack.

#### Scenario: Decode from arbitrary bytes
- **WHEN** a consumer has raw JSON bytes for a supported entity (obtained via any transport)
- **THEN** the client's model types can decode that JSON directly, without invoking the client's network methods

### Requirement: Equivalent network methods
Each client SHALL provide the same category of network methods as the Rust client (list and get operations per entity, e.g. `League.list`, `Match.get`) targeting the same OpenLigaDB endpoints.

#### Scenario: Listing leagues over the network
- **WHEN** a consumer calls the Swift or Go client's league list method
- **THEN** the client issues a request to OpenLigaDB's `getavailableleagues` endpoint and returns a collection of decoded League models, or an error if the request fails

#### Scenario: Network failure surfaces as an error
- **WHEN** an OpenLigaDB request fails (network error or non-success response)
- **THEN** the client's network method returns an error/failure result rather than throwing an unrecoverable exception or returning a partially-decoded model

### Requirement: Injectable transport for testing
Each client SHALL allow its default HTTP transport to be substituted, so tests can exercise model decoding and request construction without making real network calls.

#### Scenario: Test substitutes a fake transport
- **WHEN** a test configures the client with a fake/mock transport that returns canned responses
- **THEN** the client's network methods use that transport instead of performing a real HTTP request
