## Why

`FullTime.rs` (this umbrella's `Apps/rust`) and `dtrpg-app.rs` (a sibling desktop app in a
different project) independently built the same GPUI desktop-app scaffolding: a `core`/`ui`
crate-split workspace, light/dark theming, `rust-i18n` localization, a persistent header + status
bar shell with activity/alerts-style utility buttons, `tracing`-based logging with optional
Sentry, `cargo-packager` metadata, and CI/dependabot setup matching this org's
`repo-setup-standard` conventions. Neither app's scaffolding was built for reuse, so the next new
GPUI app (and any future refactor of this shared shape) starts from a copy-paste of whichever app
is closest, drifting further from both over time. A template repo extracts the genuinely
reusable, product-agnostic slice once.

## What Changes

- Create a new GitHub repository (a GitHub **template repository**, using the platform's native
  "Use this template" mechanism — see `design.md` for why over `cargo-generate`), seeded from the
  shared scaffolding both `FullTime.rs` and `dtrpg-app.rs` already contain: workspace layout,
  theming, i18n, shell (header/status bar), logging + optional Sentry, `cargo-packager` metadata,
  and the org's standard CI/release workflows + dependabot config.
- Placeholder-based renaming convention (product name, bundle identifier, crate names) a
  consumer resolves after clicking "Use this template", documented in the new repo's own
  `SETUP.md`.
- **Explicitly excluded** (not genericized into the template): `FullTime.rs`'s WASM plugin host
  runtime (`plugin-host-runtime`) and any league-data/catalog-domain code from either app;
  `dtrpg-app.rs`'s vendored full Lucide icon set (1994 SVGs) is not copied wholesale — the
  template ships only the handful of icons the shell itself uses, documenting how to vendor more.
- No existing repo's behavior changes. This is a new, standalone deliverable; `FullTime.rs` and
  `dtrpg-app.rs` are read as reference material, not modified.

## Capabilities

### New Capabilities

- `rust-gui-app-template`: the template repository's own scaffolding contract — what it ships,
  what a consumer must rename/configure after generating from it, and what it deliberately
  excludes.

### Modified Capabilities

(none — no existing repo's specs change)

## Impact

- **New GitHub repository** (name TBD in `design.md`) under the `pilgrimagesoftware` org,
  configured as a template repository.
- **No change to `FullTime.rs`, `dtrpg-app.rs`, or any other existing repo.** Both are read-only
  reference sources for this work.
- **`repo-setup-standard`** (the existing skill for scaffolding CI/dependabot/community docs on a
  *new* repo) is complementary, not overlapping: that skill assumes a repo's product-specific code
  already exists and adds infra around it; this template provides the product-agnostic GPUI
  app shell itself, for a repo that doesn't exist yet.
