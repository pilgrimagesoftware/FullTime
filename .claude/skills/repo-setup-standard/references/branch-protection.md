# Branch protection

Apply identically to `master` and `develop`. Write the body to a file first (`gh api` reads it
via `--input`), then PUT it to both branches:

```bash
cat > /tmp/branch-protection.json <<'EOF'
{
  "required_status_checks": {
    "strict": true,
    "contexts": ["ci / Validate on Linux", "ci / Validate on macOS (Apple Silicon)"]
  },
  "enforce_admins": false,
  "required_pull_request_reviews": null,
  "restrictions": null,
  "allow_force_pushes": false,
  "allow_deletions": false
}
EOF

gh api --method PUT repos/<owner>/<repo>/branches/master/protection --input /tmp/branch-protection.json
gh api --method PUT repos/<owner>/<repo>/branches/develop/protection --input /tmp/branch-protection.json

rm /tmp/branch-protection.json
```

## Where the required context names come from

The reusable workflow (`pilgrimagesoftware/github-actions/.github/workflows/rust-ci.yaml`) names
its job `Validate on ${{ matrix.entry.name }}`, with a default matrix of Linux and Apple Silicon
macOS. A caller workflow that invokes it from a job with id `ci` (as in `Libs/plugin-api`'s
`.github/workflows/ci.yaml`) produces the combined status check context `ci / Validate on
<entry-name>`.

If the repo's `ci.yaml` uses a different job id, or the reusable workflow's default matrix
changes, the context names change too — read the actual `ci.yaml` in the target repo and the
current `rust-ci.yaml` in `pilgrimagesoftware/github-actions` rather than assuming these two
literal strings always apply.

## Verifying what's already set

```bash
gh api repos/<owner>/<repo>/branches/master/protection
gh api repos/<owner>/<repo>/branches/develop/protection
```

Useful when a repo already has partial branch protection and you want to confirm what changed
rather than blindly overwriting it — a PUT to this endpoint replaces the whole protection
config, it doesn't merge.
