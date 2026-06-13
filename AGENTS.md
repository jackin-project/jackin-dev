# AGENTS.md

This is a Claude Code plugin providing development workflow skills for the [jackin](https://github.com/jackin-project/jackin) project.

All skills are **manual-only** — the operator invokes each explicitly as `/jackin-dev:<name>`. None auto-fire (each `SKILL.md` sets `disable-model-invocation: true`).

## Available Skills

### propose

Use when the operator runs `/jackin-dev:propose`.

Opens a feature/idea as a roadmap item draft plus an early PR, without writing code. Scaffolds `docs/content/docs/reference/roadmap/<slug>.mdx` (Problem/Why/Design/Tasks/Related Files) + sidebar entry, then stops.

Skill definition: `skills/propose/SKILL.md`

### brainstorm

Use when the operator runs `/jackin-dev:brainstorm`.

Freeform design discussion that writes decisions incrementally into a roadmap item's `## Design`. Never writes code.

Skill definition: `skills/brainstorm/SKILL.md`

### research

Use when the operator runs `/jackin-dev:research`.

Produces a standalone multi-page research dossier under `docs/content/docs/research/<slug>/` (web via `deep-research` + codebase). Brief-driven; executed via `/goal Follow <brief>`.

Skill definition: `skills/research/SKILL.md`

### create-pr

Use when the operator runs `/jackin-dev:create-pr`.

Opens a small-fix PR: branch, inline commit, body built from `.github/PULL_REQUEST_TEMPLATE.md` with verify-locally blocks auto-selected from the diff. Also the shared PR-mechanics path.

Skill definition: `skills/create-pr/SKILL.md`

### merge-pr

Use when the operator runs `/jackin-dev:merge-pr`.

Runs the pre-merge gate fail-closed (blast-radius confirm, poll CI until green, roadmap retirement, metadata reconcile) and squash-merges with `(#N)` + trailers.

Skill definition: `skills/merge-pr/SKILL.md`

### release-check

Use when preparing for a release or verifying release readiness.

Runs pre-release validation checks: CI status, local tests, clippy, fmt, doc link validation, TODO freshness, security exceptions review, and Docker build status.

Skill definition: `skills/release-check/SKILL.md`

### release-notes

Use when generating or updating the changelog, preparing release notes, or populating the Unreleased section of CHANGELOG.md.

Generates the `[Unreleased]` section of `CHANGELOG.md` from merged PRs since the last tag, classified into Keep a Changelog categories.

Skill definition: `skills/release-notes/SKILL.md`

### release

Use when performing a release, cutting a new version, or running the full release process.

Orchestrates the full release flow: runs release-check, generates release notes, recommends version, gets user confirmation, then runs `cargo release`.

Skill definition: `skills/release/SKILL.md`

## Requirements

- The target repository must have `cargo-release` configured with `release.toml`
- The target repository must have a `CHANGELOG.md` with `<!-- next-header -->` marker
- The `gh` CLI must be authenticated
- A `.github/workflows/ci.yml` workflow must exist for CI status checks

## Commit Messages

All commits in this repository MUST follow [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/).

Subject format: `<type>[optional scope][!]: <description>`

Allowed types:

| Type       | Use for                                                |
| ---------- | ------------------------------------------------------ |
| `feat`     | New user-visible feature                               |
| `fix`      | Bug fix                                                |
| `docs`     | Documentation-only change                              |
| `style`    | Formatting, whitespace; no logic change                |
| `refactor` | Internal restructuring; no behavior change             |
| `perf`     | Performance improvement                                |
| `test`     | Adding or updating tests                               |
| `build`    | Build system, tooling, dependencies                    |
| `ci`       | CI configuration                                       |
| `chore`    | Routine maintenance (release, merge, deps)             |
| `revert`   | Reverts a prior commit                                 |

Scope is optional but encouraged when it clarifies the change area.

Breaking changes use `!` after the type/scope (`feat!:` or `feat(api)!:`) and include a `BREAKING CHANGE:` footer in the body.

PR squash-merge: the PR title becomes the commit subject, so PR titles must also follow this convention.
