## Context

Two independently-built GPUI desktop apps in different projects — `FullTime.rs` (`Apps/rust` in
this umbrella) and `dtrpg-app.rs` (DriveThruRPG's desktop app) — converged on the same shape by
coincidence, not shared code:

| Concern | `FullTime.rs` | `dtrpg-app.rs` |
|---|---|---|
| Workspace | `crates/fulltime-core` + `crates/fulltime-ui`, same lints | `crates/dtrpg-core` + `crates/dtrpg-ui`, same lints |
| GUI framework | `gpui` + `gpui-component`, same pinned `rev` | `gpui` + `gpui-component`, same pinned `rev` |
| Theming | Light/dark `FullTimeTheme`, token-based colors | Light/dark, token-based colors (more theme variants) |
| i18n | `rust-i18n`, YAML locale files | `rust-i18n`, YAML locale files |
| Shell | Persistent header + status bar (disclaimer, utility buttons) | Persistent header + status bar (library summary, activity/alerts buttons) |
| Logging | `tracing` + optional `sentry` feature | `tracing` (Sentry not yet confirmed) |
| Packaging | `cargo-packager` metadata in `Cargo.toml` | Custom `package.yaml`/`release.yaml` GitHub Actions building installers |
| CI | Calls reusable workflows in `pilgrimagesoftware/github-actions` | Inline, repo-owned workflows |

Both are otherwise filled with product-specific code (`FullTime.rs`'s WASM plugin host, league
data screens; `dtrpg-app.rs`'s catalog/library, downloads, DriveThruRPG SDK integration) that has
no place in a template.

## Goals / Non-Goals

**Goals:**
- A new GitHub template repository a developer can click "Use this template" on and get a
  building, running (empty-content) GPUI desktop app: themed shell, i18n wired, logging, CI,
  packaging — in minutes, not by copying and stripping down an existing product.
- One documented, scriptable renaming step (product name, bundle identifier, crate names) since
  GitHub's template mechanism does no substitution itself (unlike `cargo-generate`).
- Match this org's existing `repo-setup-standard` CI/dependabot conventions, so a repo generated
  from this template is already compliant rather than needing that skill run afterward too.

**Non-Goals:**
- Porting `dtrpg-app.rs`'s activity/alerts status-bar system, settings persistence, or keyring
  usage into the template as-is. `FullTime.rs`'s own `add-activity-and-alerts-status-bar` change
  is porting a *scoped-down* version of that pattern into `FullTime.rs` directly; once that lands
  and is validated in a second real app, it's a better candidate for the template than porting
  `dtrpg-app.rs`'s original (heavier, download-oriented) version untested a second time.
- Vendoring `dtrpg-app.rs`'s full 1994-icon Lucide set. The template ships the handful of icons
  the shell itself uses (brand mark placeholder, theme toggle, generic nav icons) and documents
  how a consumer adds more from Lucide as they need them.
- `cargo-generate` support. Considered and rejected for *this* request — the user asked for a
  GitHub template repo specifically, and `cargo-generate` would add a second required tool for
  consumers. A `scripts/rename-template.sh` gives one-command renaming without it; revisit if a
  real need for `cargo-generate`'s templating (conditional file inclusion, prompted values) shows
  up later.

## Decisions

**New repo name: `rust-gui-app-template`.** Matches the capability name and is unambiguous about
what it's for. Alternative considered: naming after GPUI specifically (`gpui-app-template`) —
rejected since the template is opinionated beyond bare GPUI (theming/i18n/shell conventions), and
"Rust GUI app" is what a potential user searches for.

**CI/release workflows follow `FullTime.rs`'s reusable-workflow convention
(`pilgrimagesoftware/github-actions`), not `dtrpg-app.rs`'s inline copies.** A template's CI is
either "already the org standard" or "one more thing to reconcile with the org standard later" —
the reusable-workflow form is the one already documented in `repo-setup-standard`, so a repo
generated from this template needs zero CI changes to be compliant. `dtrpg-app.rs`'s inline
workflows predate that convention.

**Renaming via `scripts/rename-template.sh <product-name> <bundle-id>`, not `cargo-generate`
placeholders.** The script does a straightforward find-and-replace across `Cargo.toml`s,
`Cargo.lock`'s package names, the packager bundle identifier, and the crate directory names
themselves (`crates/template-app-core` → `crates/<name>-core`), then deletes itself and prints a
reminder to review `README.md`/`SECURITY.md` placeholders manually. Crate names in the template
default to `template-app-core`/`template-app-ui` (valid, buildable Rust identifiers) rather than a
non-buildable placeholder token, so the template repo itself always builds — a consumer who
forgets to run the script gets a working (oddly-named) app, not a broken checkout.

**Theming, i18n, and the header/status-bar shell are copied from `FullTime.rs`, not
`dtrpg-app.rs`.** `FullTime.rs`'s versions are simpler (fewer theme variants, no
settings-persistence dependency) and were built more recently against the current
`gpui`/`gpui-component` pin — less drift to reconcile. `dtrpg-app.rs`'s richer status-bar pattern
(Popover-anchored panels) is still the reference for *how* to extend the shell once a real need
exists, documented in the template's own `README.md` rather than pre-built.

## Risks / Trade-offs

- [The template's shell/theming will drift from `FullTime.rs`'s as that app keeps evolving, since
  there's no ongoing sync mechanism] → Accepted: a template is a starting point, not a shared
  dependency; revisit only if drift becomes a real, repeatedly-hit problem (e.g. via a documented
  "diff against the template" habit before major shell changes in either direction).
- [Manual rename script could miss a spot (e.g. a hardcoded string in a doc comment)] → The script
  prints an explicit post-run checklist (README, SECURITY.md, LICENSE copyright holder) rather
  than silently claiming completeness.

## Migration Plan

1. Create the new GitHub repository, empty, configured as a template repository (org settings,
   not code).
2. Scaffold its workspace from `FullTime.rs`'s current shape (core/ui split, lints, `gpui`/
   `gpui-component` pins), stripping all league-data/plugin-host-specific code.
3. Port theming, i18n, and shell (header + status bar) from `FullTime.rs`, genericizing brand
   references (app name, icon) into the `template-app-*` placeholder names.
4. Add `cargo-packager` metadata and the reusable CI/release workflow callers, dependabot config,
   and community docs, following `repo-setup-standard`.
5. Write `scripts/rename-template.sh` and `SETUP.md` documenting the post-generation steps.
6. Validate end-to-end: generate a throwaway repo from the template, run the rename script, confirm
   it builds, runs, and passes CI.

No rollback concern — this is a new, standalone repo with no consumers yet.

## Open Questions

- Should Sentry support (present in `FullTime.rs`, unclear in `dtrpg-app.rs`) be included as an
  optional feature in the template, or left for a consumer to add? Default to including it
  (already generic/product-agnostic in `FullTime.rs`'s implementation), but confirm during
  implementation that it introduces no FullTime-specific assumptions (DSN env var naming, etc.)
  that need genericizing too.
