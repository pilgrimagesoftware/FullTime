## ADDED Requirements

### Requirement: Standard Data-Provider Interface
Every data-provider plugin SHALL implement a common WIT interface exposing operations to list competitions, fetch fixtures, fetch results, fetch standings, and fetch team/competition metadata.

#### Scenario: Host queries a plugin's supported operations
- **WHEN** the host inspects a loaded plugin's exported interface
- **THEN** the plugin reports which of the standard data-provider operations it implements

#### Scenario: Host calls an operation the plugin implements
- **WHEN** the host invokes `fetch_fixtures` on a plugin that implements it
- **THEN** the plugin returns fixture data conforming to the canonical `league-data-schema`

### Requirement: Canonical Schema Output
A plugin SHALL map its upstream source data to the canonical `league-data-schema` before returning it to the host; the host SHALL NOT receive or interpret any plugin-specific raw format.

#### Scenario: Plugin returns data in canonical schema
- **WHEN** a plugin successfully fetches fixtures from its upstream source
- **THEN** the data returned to the host validates against the canonical fixture schema

#### Scenario: Plugin cannot map upstream data to the schema
- **WHEN** a plugin receives upstream data it cannot represent in the canonical schema
- **THEN** the plugin returns a structured error to the host rather than partial or malformed schema data

### Requirement: Error Reporting
A plugin SHALL report upstream failures (network errors, unexpected upstream response formats, rate limiting) to the host as structured errors rather than causing an unhandled trap.

#### Scenario: Upstream source is unreachable
- **WHEN** a plugin's upstream HTTP call fails due to a network error
- **THEN** the plugin returns a structured error identifying the failure category to the host, and the host can distinguish it from a successful empty result

#### Scenario: Upstream source rate-limits the plugin
- **WHEN** a plugin's upstream source responds with a rate-limit error
- **THEN** the plugin surfaces a rate-limit-specific structured error the host can use to back off and retry later

### Requirement: Interface Versioning
The data-provider interface SHALL be versioned so the host can detect and reject plugins built against an incompatible interface version before invoking them.

#### Scenario: Plugin built against a newer interface than the host supports
- **WHEN** the host loads a plugin declaring an interface version newer than any version the host implements
- **THEN** the host refuses to load the plugin and reports a version-incompatibility error
