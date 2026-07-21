# Branch protection

The org standard is **GitHub rulesets** (`/repos/<owner>/<repo>/rulesets`), not the older
classic branch-protection API (`/repos/<owner>/<repo>/branches/<branch>/protection`). Confirmed
by reading `dtrpg-sdk.rs`'s live config — a Rust SDK crate on the same `master`/`develop`
git-flow as the repo you're scaffolding, so its ruleset shape is the reference:

```bash
gh api repos/pilgrimagesoftware/dtrpg-sdk.rs/rulesets --jq '.[].id' | while read -r id; do
  gh api repos/pilgrimagesoftware/dtrpg-sdk.rs/rulesets/$id
done
```

Re-run that read before scaffolding a new repo — this file documents the shape as of when it
was written, not a live source of truth.

## Three rulesets

**1. `Code Quality Copilot review for default branch`** — targets `~DEFAULT_BRANCH` (a
GitHub-resolved token, not a literal branch name). One rule: `copilot_code_review` with
`review_on_push: true`, `review_draft_pull_requests: true`.

**2. `develop`** — targets `refs/heads/develop`. Rules: `deletion`, `copilot_code_review`
(`review_on_push: true`, `review_draft_pull_requests: false`), `required_status_checks` (the
`ci / Validate on ...` contexts — see below), `code_scanning` (CodeQL,
`security_alerts_threshold`/`alerts_threshold: high_or_higher`/`errors`), `code_quality`
(`severity: errors`). Bypass actors: `OrganizationAdmin` (`actor_id: null`, `bypass_mode:
always`) and the `Integration` actor for the PSW CI GitHub App (`actor_id: 4234816`,
`bypass_mode: always`) — **this bypass is required**, not decorative: `rust-release.yaml`'s
merge-back step pushes directly to `develop` after a release, and without it that push is
blocked by this same ruleset.

**3. `master`** — targets `refs/heads/master`. Rules: `deletion`, `code_scanning`,
`code_quality`, `copilot_code_review` (`review_draft_pull_requests: false`),
`required_signatures`. Note what's *not* here: no `required_status_checks` on `master` in the
reference config — `master` only receives already-CI-validated `release/*` merges, so the
gate lives on `develop`, not duplicated here. Bypass actors: `OrganizationAdmin` only.

## Status check context names

`required_status_checks` contexts come from the reusable `rust-ci.yaml`'s job name (`Validate
on ${{ matrix.entry.name }}`), prefixed by the calling `ci.yaml`'s job id. With a job id of
`ci` (as in this skill's standard `ci.yaml`), that's:

```
ci / Validate on Linux
ci / Validate on macOS (Apple Silicon)
```

Don't invent different names, and don't copy `dtrpg-sdk.rs`'s literal context strings without
checking — that repo's contexts carry an `integration_id` (a third-party check-runner specific
to that repo), which a plain GitHub Actions-based `ci.yaml` doesn't need or have.

## Creating the rulesets

`gh api` needs each ruleset body as JSON via `--input`. Example for `develop`:

```bash
cat > "$TMPDIR/ruleset-develop.json" <<'EOF'
{
  "name": "develop",
  "target": "branch",
  "enforcement": "active",
  "conditions": { "ref_name": { "include": ["refs/heads/develop"], "exclude": [] } },
  "bypass_actors": [
    { "actor_id": null, "actor_type": "OrganizationAdmin", "bypass_mode": "always" },
    { "actor_id": 4234816, "actor_type": "Integration", "bypass_mode": "always" }
  ],
  "rules": [
    { "type": "deletion" },
    { "type": "copilot_code_review", "parameters": { "review_on_push": true, "review_draft_pull_requests": false } },
    {
      "type": "required_status_checks",
      "parameters": {
        "strict_required_status_checks_policy": true,
        "do_not_enforce_on_create": false,
        "required_status_checks": [
          { "context": "ci / Validate on Linux" },
          { "context": "ci / Validate on macOS (Apple Silicon)" }
        ]
      }
    },
    { "type": "code_scanning", "parameters": { "code_scanning_tools": [
      { "tool": "CodeQL", "security_alerts_threshold": "high_or_higher", "alerts_threshold": "errors" }
    ] } },
    { "type": "code_quality", "parameters": { "severity": "errors" } }
  ]
}
EOF
gh api --method POST repos/<owner>/<repo>/rulesets --input "$TMPDIR/ruleset-develop.json"
```

Repeat with the `master` and default-branch bodies (see the rule lists above). Delete a wrong
ruleset with `gh api --method DELETE repos/<owner>/<repo>/rulesets/<id>` before re-creating —
there's no in-place "replace" for the whole object via `gh api`, only per-field `PATCH`.

## Verifying what's already set

```bash
gh api repos/<owner>/<repo>/rulesets
gh api repos/<owner>/<repo>/rulesets/<id>
```

A repo can also have leftover classic branch protection (`/branches/<branch>/protection`) from
before it adopted rulesets — check that endpoint too and delete it
(`gh api --method DELETE repos/<owner>/<repo>/branches/<branch>/protection`) if found, so the
two mechanisms don't silently stack.
