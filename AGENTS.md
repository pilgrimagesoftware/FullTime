# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code), Codex (openai.com/codex/), GitHub Copilot (copilot.github.com), and other models when working with code in this repository.

## About This Project

This is the master meta-repository for the Football App (Fussballergebnisse) project, which organizes 
SDK libraries and desktop applications for display information about football matches. The project uses 
a nested submodule architecture to manage multiple language implementations.

## OpenSpec Guidance

OpenSpec is initialized in the top-level repo and in selected child repositories.

- Use this repository (`Fussballergebnisse`) for umbrella changes that coordinate work across multiple 
  repositories.
- Use `app` for the Rust desktop application implementation.
- Use `openliga.db` for the API for the Football App (Fussballergebnisse) project.
- Update this document (`AGENTS.md`) as needed to reflect changes in the project, such as additional
  API libraries.

Current preferred example chain:

- Top-level umbrella: `Fussballergebnisse/openspec/changes/improve-auth-session-lifecycle`
- Library child: `openliga.db/openspec/changes/define-auth-session-contract`
# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code), Codex (openai.com/codex/), GitHub Copilot (copilot.github.com) when working with code in this repository.

## About This Project

This is the master meta-repository for the DriveThruRPG project, which organizes SDK libraries and desktop
applications for interacting with the DriveThruRPG API. The project uses a nested submodule architecture to
manage multiple language implementations.

## OpenSpec Guidance

OpenSpec is initialized in the top-level repo and in selected child repositories.

- Use this repository (`dtrpg`) for umbrella changes that coordinate work across multiple repositories.
- Use `dtrpg-api` for API contract changes that SDKs and applications depend on.
- Prefer `dtrpg-sdk/rust` as the language-specific downstream example when following an existing OpenSpec pattern.
- Keep `dtrpg-sdk/swift` in place as an additional reference, but prefer Rust for new example-driven SDK OpenSpec work.

Current preferred example chain:

- Top-level umbrella: `dtrpg/openspec/changes/improve-auth-session-lifecycle`
- API contract child: `dtrpg-api/openspec/changes/define-auth-session-contract`
- Preferred SDK child: `dtrpg-sdk/rust/openspec/changes/define-rust-auth-session-behavior`

## Repository Structure

This repository contains two repositories as git submodules:

- **openliga.db** - SDK implementation in Rust for interacting with the OpenLigaDB API

- **app** - Desktop application implementation in Rust and Tauri

## Working with Submodules

### Initial Setup

```bash
# Clone with all nested submodules
git clone --recursive git@github.com:pilgrimagesoftware/dtrpg.git

# Or initialize submodules after cloning
git submodule update --init --recursive
```

### Updating Submodules

```bash
# Update all submodules to latest commits on their tracked branches
git submodule update --remote --merge

# Update specific submodule
git submodule update --remote --merge openliga.db

# Update nested submodules within openliga.db
cd openliga.db && git submodule update --remote --merge
```

### Checking Submodule Status

```bash
# Check status of direct submodules
git submodule status

# Check nested submodules
cd openliga.db && git submodule status
cd app && git submodule status
```

### Making Changes

When making changes to submodules:
1. Navigate into the submodule directory
2. Make changes and commit within the submodule
3. Push the submodule changes to its remote
4. Return to the parent repository and commit the updated submodule reference
5. Push the parent repository changes

```bash
cd openliga.db
# Make changes, commit, push
git add . && git commit -m "Update openliga.db SDK"
git push

cd /Users/paulyhedral/Code/Fussballergebnisse/openliga.db
git add openliga.db
git commit -m "Update openliga.db submodule"
git push
```

## Repository Branches and Workflow

All meta-repositories use the `master` branch as the source of truth.

All code repositories use the `develop` branch as the source of truth, with the "Git Flow" process for
branch creation and merging.

## Writing Code and Generating Files

- YAML files should always use the `.yaml` extension.
- Code should always include doc comments.

## Commit Messages

Commit messages follow the "Conventional Commits" format: https://www.conventionalcommits.org/en/v1.0.0/

## Architecture Notes

- **Meta-repository Pattern**: This is a top-level organizational repository. Actual development happens in the
  submodules.
- **Language-Specific SDKs**: Each language SDK (Go, Python, Rust, Swift) is maintained as a separate repository,
  allowing independent versioning and release cycles.
- **Shared API Documentation**: The `dtrpg-sdk/api` submodule contains the source of truth for API specifications
  used across all SDK implementations.
- **Nested Submodules**: Both `dtrpg-sdk` and `dtrpg-app` are meta-repositories themselves containing their own
  submodules. Use `--recursive` flags when appropriate.

## Common Issues

### Detached HEAD State

Submodules often end up in detached HEAD state. Before making changes, ensure you're on a branch:

```bash
cd <submodule-directory>
git checkout master  # or appropriate branch
```

### Submodule Not Initialized

If a submodule directory is empty or shows errors:

```bash
git submodule update --init --recursive
```
