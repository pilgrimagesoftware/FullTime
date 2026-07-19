## Context

The Rust desktop app (`Apps/rust`) currently depends directly on `Libs/openligadb`, a Bundesliga-specific SDK, to fetch fixtures, results, and standings. Adding EPL, national team competitions, or any further league means either extending `openliga.db` with unrelated APIs or writing another bespoke integration wired into the app the same way. Neither scales, and both force a full app release for every new data source.

Zed's extension model is the reference point: extensions are compiled to WASM, loaded into a sandboxed runtime inside the host application, and communicate with the host through a narrow, versioned interface rather than linking directly into the host binary. This lets the host ship independently of extensions and lets extensions be added, updated, or removed without a host rebuild.

## Goals / Non-Goals

**Goals:**
- Let a league/competition data source be added, updated, or removed without modifying or rebuilding the desktop app.
- Sandbox plugin code so a misbehaving or malicious plugin cannot access the filesystem, network, or app state beyond what the host explicitly grants.
- Give the app a single canonical data shape to render, regardless of which plugin produced it.
- Support first-party plugins (Bundesliga, EPL, national teams) built and reviewed by the project, with room for third-party plugins later.

**Non-Goals:**
- A public third-party plugin marketplace/store (future work; this change only requires the API to be stable enough to make that possible later).
- Plugins that render UI directly (Zed extensions can contribute UI; this change scopes plugins to data supply only).
- Real-time push/streaming data (initial API is pull/poll-based, matching current `openliga.db` usage).

## Decisions

**WASM runtime: `wasmtime` with the Component Model (WIT), not raw `wasm32-unknown-unknown` + hand-rolled ABI.**
Rationale: the Component Model gives typed, versioned interfaces (WIT) instead of a bespoke serialization protocol over linear memory, which is what Zed's extension API is itself built on. Hand-rolling an ABI is the largest source of plugin-breakage-on-host-upgrade risk; WIT's interface versioning avoids that. Alternative considered: `wasmer` — comparable runtime, but `wasmtime` has stronger Component Model maturity and is the reference implementation.

**Plugins do not get direct network access; the host provides an HTTP capability.**
Rationale: sandboxing means plugin WASM modules cannot open sockets. The host exposes a constrained `fetch(request) -> response` host function (WASI HTTP, or a minimal custom host function if WASI HTTP proves too heavy) so plugins can call their upstream API without being granted raw network access. This matches Zed's model of host-provided capabilities rather than ambient authority. Alternative considered: give plugins direct WASI socket access — rejected, too broad a trust boundary for third-party code.

**Canonical schema is versioned and owned by the host, not by any plugin.**
Rationale: `league-data-schema` (fixtures, results, standings, teams, competitions) must be stable so the UI never special-cases a plugin. Plugins map their source format to this schema at the plugin/host boundary. Alternative considered: let each plugin define its own schema and have the app adapt per-plugin — rejected, this is exactly the coupling the plugin system exists to remove.

**Plugin manifest is a static TOML/JSON file declaring identity, version, declared capabilities (network hosts it needs to call), and the schema version it targets.**
Rationale: the host must be able to decide whether to load a plugin (compatible schema version, acceptable requested capabilities) without executing it. Mirrors Zed's `extension.toml`.

**`openliga.db` becomes the reference implementation of the plugin API, not a special-cased built-in.**
Rationale: proves the API is sufficient for a real data source and prevents the host from silently depending on non-plugin code paths for its primary data source. If the reference plugin can't be expressed through the API, the API is wrong.

## Risks / Trade-offs

- [WASM component call overhead vs. the current in-process crate call] → Data-provider calls are network-bound (HTTP fetch dominates), so marshaling overhead is expected to be negligible; validate with a benchmark comparing current `openliga.db` in-process calls against the plugin path before this ships.
- [WASI HTTP / Component Model tooling is still stabilizing in the Rust ecosystem] → Pin `wasmtime` and `wit-bindgen` versions explicitly; if WASI HTTP proves unworkable, fall back to a minimal custom host-provided `fetch` function instead of the standard WASI HTTP interface, at the cost of losing some ecosystem interop.
- [Breaking change to how the app sources Bundesliga data] → Ship the Bundesliga plugin and validate it against current `openliga.db` output before removing the direct dependency, so the migration is a swap, not a rewrite.
- [Third-party plugins are a future goal but the sandbox/capability model must be right from day one] → Design the manifest's capability declarations (network hosts) and the host's enforcement of them now, even though only first-party plugins ship initially, rather than retrofitting sandboxing later.
- [Canonical schema won't fit every competition's real data shape (e.g., national team qualifiers with group stages vs. league tables)] → Model the schema around the union of Bundesliga + EPL + at least one national-team competition during design, not just the first plugin.

## Migration Plan

1. Define the WIT interface and canonical schema; build the plugin host runtime in `Apps/rust` behind a feature flag, without removing the existing `openliga.db` integration.
2. Build the Bundesliga plugin against `openliga.db`'s existing API surface, wrapping it rather than rewriting its HTTP logic.
3. Run the plugin path and the direct-integration path side by side, diffing output against the canonical schema to validate equivalence.
4. Cut the app over to the plugin path for Bundesliga; remove the direct `openliga.db` dependency from the app.
5. Ship the plugin SDK/template and build the EPL and a national-team competition plugin as the second and third proof points of the API's generality.

Rollback: keep the feature flag until step 4 is validated in production use; reverting is disabling the flag and re-enabling the direct `openliga.db` dependency, which is not removed until step 4.

## Open Questions

- Does the app need to support hot-reloading/updating a plugin without an app restart, or is restart-to-update acceptable for v1?
- Where do plugin binaries live — bundled in the app installer for first-party plugins, downloaded at runtime, or both?
- What's the update/compatibility policy when the canonical schema needs a breaking change — versioned schema negotiation, or a hard cutover requiring all plugins to update together?
