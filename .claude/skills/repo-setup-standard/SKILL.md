---
name: repo-setup-standard
description: Scaffold a new or under-scaffolded pilgrimagesoftware Rust repo (a new plugin, SDK language, or app repo) with the org's standard CI/release workflows, dependabot, branch protection, community docs, and AGENTS.md/CLAUDE.md robot setup. Use whenever the user asks to "set up the repo", "add repo scaffolding", "bootstrap CI", or start a brand-new repo before implementation begins.
metadata:
  type: engineering
---

# Repo setup standard

Bring a repo up to the org's current baseline: CI/release workflows, dependabot, branch
protection, community docs, and robot guidance. This is infrastructure work, separate from
implementing whatever the repo actually does — do this first, commit it on its own, then move
to feature work.

**Source of truth: `Libs/plugin-api` in the `FullTime` umbrella repo.** It's the newest sibling
and the pattern every new repo should copy. Do not copy `Libs/openligadb/rust` — it predates the
current pattern and uses a different, retired CI/release approach (`actions-rs/cargo`,
`anothrNick/github-tag-action`). If `docs/git-flow.md` in the umbrella repo ever describes a
release mechanism that disagrees with what `plugin-api` actually has committed, trust
`plugin-api` — the doc describes the general git-flow shape, not the org's actual reusable-workflow
wiring, which lives in the sibling repo's real files.

Read the target files directly out of `Libs/plugin-api` in the FullTime umbrella checkout before
writing anything — this skill describes what to create and why, not the literal bytes, and
`plugin-api` may have moved on since this was written.

## What to add

1. **CI/release workflows** (`.github/workflows/`) — four thin wrappers around reusable
   workflows in `pilgrimagesoftware/github-actions@master`: `ci.yaml` (calls `rust-ci.yaml`),
   `prepare-release.yaml` (calls `rust-prepare-release.yaml`), `release.yaml` (calls
   `rust-release.yaml`), `tag-release.yaml` (calls `rust-tag-release.yaml`). Copy the four files
   from `Libs/plugin-api/.github/workflows/` verbatim except `prepare-release.yaml`'s
   `project-number` input — see step 3 below. Each caller declares an explicit top-level
   `permissions:` block matching what its called reusable workflow needs (`contents: read` for
   `ci.yaml`; `contents: write` + `pull-requests: write` for `prepare-release.yaml`;
   `contents: write` + `id-token: write` for `release.yaml`; `contents: write` for
   `tag-release.yaml`) — CodeQL's `actions/missing-workflow-permissions` flags any of the four
   that omits this.

2. **Dependabot** (`.github/dependabot.yaml`) — cargo + github-actions ecosystems, both targeting
   `develop`, weekly. Copy from `Libs/plugin-api/.github/dependabot.yaml` verbatim.

3. **Changelog and toolchain config** — `cliff.toml` (git-cliff config) and
   `rust-toolchain.toml` (pins `stable` + `rustfmt`/`clippy`). Copy both from `Libs/plugin-api`
   verbatim; they're not repo-specific.

4. **Community docs** — `CODE_OF_CONDUCT.md` (copy verbatim, it's generic), `CONTRIBUTING.md`,
   `SECURITY.md`, `RELEASING.md`, `README.md`. Copy each from `Libs/plugin-api` and adapt the
   repo-specific parts: crate name, GitHub URL, what the repo actually does, its dependency
   relationships, and (in `SECURITY.md`) what's explicitly out of scope because it belongs to a
   different repo in the plugin/SDK chain. Seed `CHANGELOG.md` with just an `[Unreleased]`
   section noting the scaffolding work.

5. **Robot guidance** — write `AGENTS.md` following the shape in
   `Libs/openligadb/rust/AGENTS.md` or `Apps/rust/AGENTS.md` (About This Project, Repository,
   Branches and Workflow, Coding Conventions, an implementation-pointers section, Running Checks
   Locally). Then symlink `CLAUDE.md` to it — a real symlink, not a copy, matching `Apps/rust`:
   `ln -s AGENTS.md CLAUDE.md`. This keeps one file to maintain instead of two drifting copies.

6. **Branch protection** — GitHub **rulesets**, not classic branch protection. `Libs/plugin-api`
   doesn't have this configured yet, so it isn't the source of truth here — `dtrpg-sdk.rs` is
   (the closest analog: another Rust SDK crate on the same `master`/`develop` git-flow). See
   `references/branch-protection.md` for the three rulesets to create and the exact `gh api`
   invocations. The required status check names (`ci / Validate on Linux`, `ci / Validate on
   macOS (Apple Silicon)`) come from the reusable `rust-ci.yaml`'s job name, prefixed by the
   calling workflow's job id (`ci`) — don't invent different names even if the repo's `ci.yaml`
   job id differs, check what it actually is first.

7. **Repository setting: allow auto-merge** — enable `Allow auto-merge` at the repo level
   (`gh repo edit <owner>/<repo> --enable-auto-merge`). This is a plain repository setting, not a
   ruleset rule, and it's off by default on new repos. Without it, `gh pr merge --auto` fails with
   `GraphQL: Auto merge is not allowed for this repository (enablePullRequestAutoMerge)` even when
   the PR is otherwise mergeable and all required checks pass — confirmed by hitting this while
   merging a PR into `openligadb.rs`'s `develop`, which had rulesets but not this setting. Verify
   with `gh api repos/<owner>/<repo> --jq '.allow_auto_merge'`.

## Before you start

Look up the repo's real GitHub Projects v2 board number rather than assuming one:

```bash
gh project list --owner pilgrimagesoftware --format json
```

Confirm which project the repo actually belongs to (ask the user if more than one is plausible),
and use that number in `prepare-release.yaml`'s `project-number` input — every repo in the org
does not necessarily share the same board.

## Gotchas

- **Branch protection checks that "never ran"**: GitHub lets you register a required status
  check context before it has ever reported — this is expected on a freshly scaffolded repo and
  isn't an error. The check starts enforcing on the first PR that runs `ci.yaml`.
- **Empty-diff PRs get rejected**: if you're also opening a PR right after scaffolding and the
  branch has no other commits, `gh pr create` fails with "No commits between X and Y" — GitHub
  won't open a PR with a zero-diff. An empty commit (`git commit --allow-empty`) unblocks it if
  a PR is genuinely needed before real work lands; don't reach for this if you can just wait for
  the first real commit.
- **Don't hardcode `project-number` or crate names** by copying them from another repo's
  workflow file — these are exactly the values that must be repo-specific (see "Before you
  start").
- **`git add`**: stage the new files explicitly by path, not `git add -A`/`.` — even though
  everything in a fresh scaffolding commit is new, an explicit list is one line of extra typing
  and never accidentally sweeps in something unrelated sitting in the worktree.
- **Auto-merge is a separate on/off repo setting, not part of any ruleset.** A repo can have
  fully correct rulesets on `develop` and still reject `gh pr merge --auto` until
  `allow_auto_merge` is turned on (step 7). Don't assume "branch protection is set up" implies
  auto-merge works.
- **Rulesets, not classic branch protection, and check a live example before guessing the
  shape.** `Libs/plugin-api` (the source of truth for everything else in this skill) has no
  branch protection of any kind configured, so it can't be copied for this step — checking it
  and finding nothing is not the same as the org having no standard. Query a same-era sibling
  repo directly (`gh api repos/pilgrimagesoftware/<repo>/rulesets`) before assuming either
  "rulesets" or "classic protection" is the pattern; `dtrpg-sdk.rs` is confirmed as of this
  writing. See `references/branch-protection.md` for what a full read of its config looks like.
