## 1. Repository Setup

- [ ] 1.1 Create the `rust-gui-app-template` GitHub repository under the `pilgrimagesoftware`
  org, empty, and enable "Template repository" in its settings
- [ ] 1.2 Run the `repo-setup-standard` skill against it once the workspace exists (task group 2)
  for CI/release workflows, dependabot, and community docs

## 2. Workspace Scaffold

- [ ] 2.1 Create the `crates/template-app-core` + `crates/template-app-ui` workspace, copying
  `FullTime.rs`'s `Cargo.toml` lints/dependency-pin structure (same `gpui`/`gpui-component` `rev`)
- [ ] 2.2 Set up the `[[bin]]`/binary entry point and `cargo-packager` metadata (bundle
  identifier, icons) with `template-app` placeholder values
- [ ] 2.3 Add `rust-toolchain.toml` matching the pinned stable channel both source apps use

## 3. Theming, i18n, and Shell

- [ ] 3.1 Port `FullTime.rs`'s light/dark theme token system (`data/theme.rs`), genericized
  (no Bundesliga/league-specific color logic)
- [ ] 3.2 Port `rust-i18n` wiring and a starter `en.yaml` locale file with the shell's own
  strings only
- [ ] 3.3 Port the persistent header (brand mark, theme toggle) and status bar shell views,
  stripped of `FullTime.rs`-specific nav (no league tabs, no screen nav beyond a placeholder)
- [ ] 3.4 Vendor only the icons the shell itself uses (brand mark placeholder, theme toggle,
  generic nav) — not `dtrpg-app.rs`'s full icon set
- [ ] 3.5 Add an empty-state placeholder content view so the generated app shows something
  beyond the header/status bar on first run

## 4. Logging and Packaging

- [ ] 4.1 Port `tracing`-based logging setup with the optional `sentry` feature, confirming no
  FullTime-specific DSN/env-var naming leaks through (see design.md's open question)
- [ ] 4.2 Confirm `cargo-packager` produces installable artifacts for macOS at minimum (Linux/
  Windows if straightforward, otherwise noted as a follow-up)

## 5. Renaming Tooling

- [ ] 5.1 Write `scripts/rename-template.sh <product-name> <bundle-id>`: renames crate
  directories, `Cargo.toml` package names, and the packager bundle identifier
- [ ] 5.2 Script prints a post-run checklist (README, SECURITY.md, LICENSE copyright holder)
  rather than claiming completeness
- [ ] 5.3 Write `SETUP.md` documenting the "Use this template" → rename → build flow end to end

## 6. Validation

- [ ] 6.1 Generate a throwaway repo from the template (via GitHub's "Use this template"),
  confirm it builds and runs with zero changes (default `template-app` branding)
- [ ] 6.2 Run `scripts/rename-template.sh` against the throwaway repo, confirm it still builds
  under the new name
- [ ] 6.3 Confirm the throwaway repo's CI passes using the org's reusable workflows
- [ ] 6.4 Delete the throwaway repo once validated
