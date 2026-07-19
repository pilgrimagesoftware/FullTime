## Why

League data currently ships as hardcoded, source-specific integrations (starting with `openliga.db` for the Bundesliga). Adding a new competition today means writing bespoke fetch/parse/normalize code inside the app itself. A plugin architecture, in the spirit of Zed's WASM extensions, lets each league or competition (Bundesliga, EPL, national teams, etc.) ship as an independently built, sandboxed, versioned module that the app loads at runtime, so new leagues can be added without touching or redeploying the core app.

## What Changes

- Introduce a WASM-based plugin host embedded in the Rust desktop app that loads, sandboxes, and manages the lifecycle of data-provider plugins.
- Define a data-provider plugin API (WIT/component interface) that plugins implement to supply fixtures, results, standings, and team/competition metadata.
- Define a canonical, source-agnostic league data schema that the app's UI and business logic consume, decoupling them from any single provider's raw format.
- Add a plugin manifest and registry format covering plugin identity, versioning, capabilities declared, and install/enable/disable/update state.
- **BREAKING**: Refactor the existing `openliga.db` integration out of the core app and re-package it as the first-party Bundesliga data-provider plugin conforming to the new API.
- Ship a plugin SDK/template so first-party (EPL, national teams) and third-party plugins can be built against a stable, documented interface.

## Capabilities

### New Capabilities
- `plugin-host-runtime`: WASM plugin loading, sandboxing, resource limits, and lifecycle management inside the desktop app.
- `data-provider-plugin-api`: the contract a plugin implements to supply league/competition data (fixtures, results, standings, teams) to the host.
- `plugin-manifest-registry`: plugin manifest format, discovery of bundled/installed plugins, versioning, and enable/disable/update state.
- `league-data-schema`: canonical normalized data model that all plugin output is mapped to, consumed by the app UI independent of data source.

### Modified Capabilities
- (none — no existing specs predate this change)

## Impact

- **Apps/rust**: new plugin host module, plugin management UI/settings, refactor of current data-fetch code paths to consume the canonical schema instead of source-specific types.
- **Libs/openligadb**: repackaged as the reference Bundesliga plugin implementation against the new plugin API rather than a direct app dependency.
- New plugin SDK crate and plugin project template for building additional providers (EPL, national teams, others).
- Documentation for plugin authors (host contract, manifest schema, versioning/compatibility policy).
