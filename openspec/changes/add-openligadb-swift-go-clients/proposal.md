## Why

`Libs/openligadb` currently ships a single language client (Rust, `openligadb.rs`) for the [OpenLigaDB API](https://www.openligadb.de/). The desktop apps in `Apps/swift` and any future Go-based tooling have no first-party client and would otherwise need to hand-roll HTTP calls and model parsing against OpenLigaDB directly. Adding Swift and Go clients that mirror the Rust client's model coverage and behavior lets those apps consume OpenLigaDB the same way the Rust app does, without duplicating parsing logic per platform.

## What Changes

- Add a new `swift` submodule under `Libs/openligadb` (`openligadb.swift`) implementing the same model surface (leagues, matches, teams, etc.) and `list`/`get` style network methods as the Rust client.
- Add a new `go` submodule under `Libs/openligadb` (`openligadb.go`) with equivalent model and client coverage.
- Both new clients follow the existing Rust client's optional-networking pattern where practical: model/serde types usable independent of the HTTP transport, network methods gated separately.
- Update `Libs/openligadb` meta-repo (`.gitmodules`, README) to register and describe the two new submodules alongside `rust`.
- Scaffold each new repository per its language's OpenSpec/CI conventions (`openspec-propose`/`repo-setup-standard` patterns already used for `rust`): `develop`/`master` git-flow branches, CI, changelog automation, README, CONTRIBUTING, LICENSE.

## Capabilities

### New Capabilities
- `openligadb-client-parity`: defines the model surface and client behavior that any OpenLigaDB client library (Rust, Swift, Go) must provide, so Swift and Go implementations stay behaviorally equivalent to the existing Rust client.

### Modified Capabilities
- (none — no existing top-level specs predate this change)

## Impact

- **Libs/openligadb**: new `swift` and `go` submodules; `.gitmodules` and top-level README updated to reference them.
- **New repositories**: `openligadb.swift` and `openligadb.go` (or equivalent names per language ecosystem convention), each scaffolded with git-flow branches, CI, and release tooling matching `openligadb.rs`.
- **Apps/swift**: unblocks consuming OpenLigaDB via a first-party Swift client instead of ad hoc networking code (no app-side changes required by this change itself).
- No changes to the existing Rust client's public API; it remains the reference implementation.
