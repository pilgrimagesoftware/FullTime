## ADDED Requirements

### Requirement: Buildable Out of the Box
A repository generated from the template SHALL build and run without requiring any
renaming step first — default crate names, product name, and bundle identifier are all
valid, non-placeholder values.

#### Scenario: Fresh generation builds immediately
- **WHEN** a consumer clicks "Use this template", clones the result, and runs the standard
  build command with no other changes
- **THEN** the build succeeds and the app launches, showing the default `template-app`
  branding

### Requirement: One-Command Rename
The template SHALL provide a script that renames the product name, bundle identifier, and
crate names throughout the repository in one command, then reports any remaining manual
steps rather than silently claiming completeness.

#### Scenario: Running the rename script
- **WHEN** a consumer runs `scripts/rename-template.sh <product-name> <bundle-id>`
- **THEN** crate directory names, `Cargo.toml` package names, and the packager bundle
  identifier are all updated to reflect the new values, and the script prints a checklist
  of files (README, SECURITY.md, LICENSE) that still need manual review

#### Scenario: Renamed repo still builds
- **WHEN** the rename script has been run
- **THEN** the repository builds successfully under its new crate names

### Requirement: Shared Shell Scaffolding
The template SHALL include a themed (light/dark) application shell — persistent header and
status bar — with `rust-i18n` localization wired end-to-end, ported from `FullTime.rs`'s
current implementation.

#### Scenario: Theme toggle works out of the box
- **WHEN** a generated app's theme toggle control is clicked
- **THEN** the shell's colors switch between the light and dark token sets

#### Scenario: Locale strings render through i18n
- **WHEN** any shell text is rendered
- **THEN** it resolves through a `t!(...)` lookup against a YAML locale file, not a
  hardcoded string

### Requirement: Org-Standard CI and Packaging
The template SHALL ship CI and release workflows that call this org's reusable workflows
(`pilgrimagesoftware/github-actions`), plus `cargo-packager` metadata, dependabot
configuration, and community docs matching the `repo-setup-standard` skill's conventions —
not `dtrpg-app.rs`'s inline, repo-owned workflow copies.

#### Scenario: Generated repo passes repo-setup-standard's expectations
- **WHEN** the `repo-setup-standard` skill's checklist is run against a freshly generated
  repository
- **THEN** CI/release workflows, dependabot, and community docs are already present and
  correctly shaped, needing no further scaffolding

### Requirement: Excludes Product-Specific Code
The template SHALL NOT include `FullTime.rs`'s WASM plugin host runtime, any league-data or
catalog-domain code from either source app, or `dtrpg-app.rs`'s full vendored icon set.

#### Scenario: No plugin host code present
- **WHEN** the template repository's source tree is inspected
- **THEN** no `wasmtime` dependency, plugin manifest format, or plugin-loading code exists
  anywhere in it

#### Scenario: Only shell-used icons are vendored
- **WHEN** the template's icon assets directory is inspected
- **THEN** it contains only the icons the shell itself renders (brand mark, theme toggle,
  generic nav), not a full icon-library dump
