# AGENTS.md

This is a Claude Code plugin providing development workflow skills for the [jackin](https://github.com/donbeave/jackin) project.

## Available Skills

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
