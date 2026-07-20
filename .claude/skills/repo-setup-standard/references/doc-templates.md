# Doc templates: what to change per repo

Every file below exists already in `Libs/plugin-api` — read it there and adapt. This file lists
only what must change per repo; everything else copies forward as-is.

## `.github/workflows/*.yaml`

Copy `ci.yaml`, `release.yaml`, `tag-release.yaml` verbatim — nothing repo-specific in them.
`prepare-release.yaml` has one line to change: `project-number` (see SKILL.md's "Before you
start").

## `.github/dependabot.yaml`, `cliff.toml`, `rust-toolchain.toml`

Copy verbatim.

## `CODE_OF_CONDUCT.md`

Copy verbatim — it's the org-wide Citizen Code of Conduct, not repo-specific.

## `CONTRIBUTING.md`

Change:
- Clone URL
- Any section describing repo-specific source-of-truth files (e.g. `plugin-api`'s WIT-contract
  section) — replace with whatever this repo's own "if you change X, also update Y" invariants
  are, or drop the section if there isn't one.

## `SECURITY.md`

Change:
- crates.io badge/link crate name
- Security Advisories URL (repo-specific)
- The "Scope" section: what this repo's crate actually covers, and — importantly — what it
  explicitly does *not* enforce because that responsibility belongs to a different repo in the
  chain (host runtime, contract-definition repo, wrapped upstream crate, etc). Naming the
  boundary here saves a report from landing in the wrong repo later.

## `RELEASING.md`

Change:
- crate name in the trusted-publishing paragraph

## `README.md`

Change everything except the section headers and the overall shape: badges (crates.io, docs.rs,
CI, license — point at this repo), the "what this does" paragraph, and any "depends on" /
"consumed by" links to sibling repos.

## `CHANGELOG.md`

Seed with just:

```markdown
## [Unreleased]

### Documentation

- Add CI/release workflows, dependabot, CODE_OF_CONDUCT, CONTRIBUTING, SECURITY, RELEASING, AGENTS, and README
```

## `AGENTS.md`

Write fresh — there's no single template repo for this one, since it should describe what makes
*this* repo distinct. Use `Libs/openligadb/rust/AGENTS.md` or `Apps/rust/AGENTS.md` as a shape
reference (both are readable examples of the section list), not a copy source. Cover: About This
Project, Repository (GitHub URL, crate name, key dependencies), Branches and Workflow, Coding
Conventions, a pointer to wherever the actual implementation contract/spec lives if this repo
implements someone else's interface, and Running Checks Locally.

Then:

```bash
ln -s AGENTS.md CLAUDE.md
```
