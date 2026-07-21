## 1. Repository scaffolding

- [ ] 1.1 Create `openligadb.swift` repository, scaffolded per `repo-setup-standard` (git-flow `master`/`develop` branches, CI, AGENTS.md/CLAUDE.md, README, CONTRIBUTING, SECURITY, CODE_OF_CONDUCT, LICENSE)
- [ ] 1.2 Create `openligadb.go` repository, scaffolded per `repo-setup-standard`, adapted to Go module conventions
- [ ] 1.3 Register both new repos as submodules under `Libs/openligadb`, update `.gitmodules`
- [ ] 1.4 Update `Libs/openligadb` top-level README to list `swift` and `go` alongside `rust`

## 2. Swift client: models

- [ ] 2.1 Define `Sport`, `League`, `Team`, `Group`, `Match`, `Goal`, `Result`, `Table`, `Location` as `Codable` structs matching the Rust client's fields and optionality
- [ ] 2.2 Map OpenLigaDB's JSON key names (e.g. `leagueId`, `leagueName`) to Swift property names via `CodingKeys`
- [ ] 2.3 Add unit tests decoding each model from fixture JSON, covering both fully-populated and fields-omitted cases

## 3. Swift client: networking

- [ ] 3.1 Define a transport abstraction (protocol) so the default `URLSession`-based transport can be substituted in tests
- [ ] 3.2 Implement list/get network methods per entity (e.g. `League.list()`, `Match.get(id:)`) against the OpenLigaDB endpoints used by the Rust client
- [ ] 3.3 Surface network/decoding failures as typed errors, not unhandled exceptions
- [ ] 3.4 Add tests using a fake transport to verify request construction and response decoding without real network calls

## 4. Go client: models

- [ ] 4.1 Define `Sport`, `League`, `Team`, `Group`, `Match`, `Goal`, `Result`, `Table`, `Location` as exported structs with `json` tags matching the Rust client's fields and optionality (pointer types or zero-value semantics for optional fields)
- [ ] 4.2 Add unit tests decoding each model from fixture JSON, covering both fully-populated and fields-omitted cases

## 5. Go client: networking

- [ ] 5.1 Define an `HTTPDoer`-style interface so the default `net/http`-based client can be substituted in tests
- [ ] 5.2 Implement list/get functions or methods per entity against the OpenLigaDB endpoints used by the Rust client
- [ ] 5.3 Surface network/decoding failures as Go errors following the client's error-wrapping conventions
- [ ] 5.4 Add tests using a fake `HTTPDoer` to verify request construction and response decoding without real network calls

## 6. Verification

- [ ] 6.1 Cross-check Swift and Go model field sets and optionality against the Rust client's `src/models/*.rs` for parity
- [ ] 6.2 Run each new repo's CI (build, lint, test) and confirm it passes
- [ ] 6.3 Confirm `Libs/openligadb` submodule pointers resolve and `git submodule status` reports clean, non-detached state for both new submodules
