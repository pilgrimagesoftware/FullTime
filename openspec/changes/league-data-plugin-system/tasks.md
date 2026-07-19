## 1. Canonical Schema

- [ ] 1.1 Define the canonical `league-data-schema` (competitions, teams, fixtures, results, standings) covering both single-table and group-based formats
- [ ] 1.2 Add a schema version identifier and document the compatibility policy for future breaking changes
- [ ] 1.3 Validate the schema against real Bundesliga, EPL, and one national-team competition data shape before finalizing

## 2. Plugin Interface Definition

- [ ] 2.1 Define the WIT interface for the data-provider plugin API (list competitions, fetch fixtures, fetch results, fetch standings, fetch metadata)
- [ ] 2.2 Define the structured error types plugins return (network failure, rate limit, schema-mapping failure)
- [ ] 2.3 Define the interface version scheme and host-side compatibility check
- [ ] 2.4 Define the plugin manifest format (ID, version, target interface/schema version, declared network hosts)

## 3. Plugin Host Runtime

- [ ] 3.1 Add `wasmtime` (Component Model) to `Apps/rust` behind a feature flag
- [ ] 3.2 Implement plugin loading/instantiation from the manifest, rejecting incompatible schema/interface versions
- [ ] 3.3 Implement the host-provided HTTP fetch capability, scoped to hosts declared in the plugin's manifest
- [ ] 3.4 Implement fault isolation so a plugin panic/trap does not crash the host or affect other plugins
- [ ] 3.5 Implement plugin unloading and re-loading without an app restart

## 4. Plugin Manifest Registry

- [ ] 4.1 Implement discovery of bundled first-party plugins at startup
- [ ] 4.2 Implement discovery of user-installed plugins from a user plugin directory
- [ ] 4.3 Implement enable/disable state per plugin, persisted across restarts
- [ ] 4.4 Implement installed-version tracking and update-availability detection for plugins with a declared update source

## 5. Bundesliga Reference Plugin

- [ ] 5.1 Scaffold the Bundesliga plugin crate wrapping `Libs/openligadb`'s existing fetch logic
- [ ] 5.2 Implement mapping from `openliga.db`'s response format to the canonical schema
- [ ] 5.3 Write the plugin manifest declaring `openliga.db`'s API host as the sole network capability
- [ ] 5.4 Run the plugin path and the existing direct integration side by side, diffing output against the canonical schema

## 6. App Cutover

- [ ] 6.1 Switch the app's UI/business logic to consume the canonical schema instead of `openliga.db` types
- [ ] 6.2 Cut the app over to loading Bundesliga data through the plugin path
- [ ] 6.3 Remove the direct `openliga.db` dependency and the feature flag from `Apps/rust`
- [ ] 6.4 Add a basic plugin management UI (list installed plugins, enable/disable, show update availability)

## 7. Plugin SDK and Additional Plugins

- [ ] 7.1 Publish a plugin SDK crate and project template documenting the host contract and manifest schema
- [ ] 7.2 Build the EPL data-provider plugin against a suitable EPL data source
- [ ] 7.3 Build a national-team competition data-provider plugin, validating the group-stage standings/fixture schema paths
- [ ] 7.4 Write plugin-author documentation covering the WIT interface, manifest format, and versioning/compatibility policy

## 8. Verification

- [ ] 8.1 Benchmark plugin-path data fetch latency against the prior in-process `openliga.db` calls
- [ ] 8.2 Test sandbox enforcement: confirm a plugin cannot access the filesystem or call an undeclared network host
- [ ] 8.3 Test fault isolation: confirm a deliberately panicking plugin does not crash the host or other loaded plugins
