# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code), Codex (openai.com/codex/), GitHub 
Copilot (copilot.github.com), and other models when working with code in this repository.

## About This Project

This is the master meta-repository for the Football App (Fussballergebnisse) project, which 
organizes SDK libraries and desktop applications for display information about football 
matches. The project uses a nested submodule architecture to manage multiple language 
implementations.

## OpenSpec Guidance

OpenSpec is initialized in the top-level repo and in selected child repositories.

- Use this repository (`Fussballergebnisse`) for umbrella changes that coordinate work across 
  multiple repositories.
- Use `app` for the Rust desktop application implementation.
- Use `openliga.db` for the API for the Football App (Fussballergebnisse) project.
- Update this document (`AGENTS.md`) as needed to reflect changes in the project, such as 
  additional API libraries.

Current preferred example chain:

- Top-level umbrella: `Fussballergebnisse/openspec/changes/improve-auth-session-lifecycle`
- Library child: `openliga.db/openspec/changes/define-auth-session-contract`

## Repository Structure

This repository contains submodules:

- **Apps** - Desktop application meta-repository
- **Data** - Data repository
- **Libs** - Libraries meta-repository

## Working with Submodules

### Initial Setup

```bash
# Clone with all nested submodules
git clone --recursive git@github.com:pilgrimagesoftware/Fussballergebnisse.git

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

All code repositories use the `develop` branch as the source of truth, with the "Git Flow" 
process for branch creation and merging.

## Writing Code and Generating Files

- YAML files should always use the `.yaml` extension.
- Code should always include doc comments.

## GitHub Issues

- When creating a new issue, use the "Bug Report" or "Feature Request" template if available.
- When creating a new issue, assign the issue to me, and add the appropriate labels and type.
- If the issue will contain multiple tasks, create sub-issues for each task as appropriate, 
  but don't make them too granular.
- When creating sub-issues, use the "Task" type and assign them to me.
- When creating sub-issues, use the appropriate labels and type.
- When creating a new issue, assign it to the "Football Scores" project.

## Commit Messages

Commit messages follow the "Conventional Commits" format: https://www.conventionalcommits.org/en/v1.0.0/

## Architecture Notes

- **Meta-repository Pattern**: This is a top-level organizational repository. Actual 
  development happens in the submodules.
- **Language-Specific SDKs**: Each language SDK (Go, Python, Rust, Swift) is maintained as a 
  separate repository, allowing independent versioning and release cycles.
- **Nested Submodules**: Both `Fussballergebnisse-Apps` and `Fussballergebnisse-Libs` are 
  meta-repositories themselves containing their own submodules. Use `--recursive` flags when 
  appropriate.

## Common Issues

### Detached HEAD State

Submodules often end up in detached HEAD state. Before making changes, ensure you're on a 
branch:

```bash
cd <submodule-directory>
git checkout master  # or appropriate branch
```

### Submodule Not Initialized

If a submodule directory is empty or shows errors:

```bash
git submodule update --init --recursive
```
