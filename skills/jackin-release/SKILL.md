---
name: jackin-release
description: Use when performing a release, cutting a new version, or running the full release process for the jackin project
argument-hint: "[major|minor|patch|version]"
disable-model-invocation: true
---

# jackin-release

Full release orchestrator for the jackin project. Runs pre-release validation, generates changelog, recommends version, and executes `cargo release`.

## When to Use

- When you want to cut a new release
- When asked to "release", "cut a version", or "ship it"

## When NOT to Use

- If you just want to check readiness: use `release-check` instead
- If you just want to update the changelog: use `release-notes` instead

## Prerequisites

- `cargo-release` must be installed: `cargo install cargo-release`
- `gh` CLI must be authenticated: `gh auth status`
- `release.toml` must have `pre-release-replacements` configured for CHANGELOG.md
- `CHANGELOG.md` must exist with `<!-- next-header -->` marker
- `.github/workflows/ci.yml` must exist

## Process

### Step 1: Run Release Check

Follow the `release-check` skill completely. Read `skills/release-check/SKILL.md` and execute all checks.

If any **blocking** check fails, **STOP**. Show the readiness report and tell the user what needs to be fixed. Do not proceed.

If only **warnings** or **review items** exist, show the report and ask: "Warnings found. Continue with release? (yes/no)"

### Step 2: Run Release Notes

Follow the `release-notes` skill completely. Read `skills/release-notes/SKILL.md` and execute all steps.

Present the generated changelog section. Allow the user to review and edit.

Do not proceed until the user approves the changelog.

### Step 3: Recommend Version

Read the approved `[Unreleased]` section from `CHANGELOG.md` and analyze the categories:

| Condition | Recommendation |
|---|---|
| Has entries in **Removed** or any entry mentions "breaking" | **major** bump |
| Has entries in **Added** | **minor** bump |
| Only has **Fixed**, **Changed**, **Security**, **Deprecated** | **patch** bump |

Get the current version:

```bash
grep '^version' Cargo.toml | head -1 | sed 's/.*"\(.*\)"/\1/'
```

Calculate the recommended next version and present:

> "Current version: v0.4.0
> Changelog has: 2 Added, 1 Fixed, 1 Changed
> Recommendation: **v0.5.0** (minor — new features added)
>
> Accept this version, or specify a different bump level? (major/minor/patch)"

Wait for user confirmation.

### Step 4: Commit Changelog

If `CHANGELOG.md` has uncommitted changes (from Step 2), commit them:

```bash
git add CHANGELOG.md
git commit -m "docs: update changelog for vX.Y.Z"
```

Replace `X.Y.Z` with the recommended version from Step 3.

### Step 5: Final Confirmation

Present a summary:

```
Release Summary
===============
Version:   v0.5.0 (minor)
Changelog: 2 Added, 1 Fixed, 1 Changed
Checks:    all green (1 warning)
Command:   cargo release minor --execute

This will:
  1. Bump version in Cargo.toml to 0.5.0
  2. Rename [Unreleased] to [0.5.0] - 2026-04-04 in CHANGELOG.md
  3. Add new [Unreleased] section
  4. Create release commit: "chore: release v0.5.0"
  5. Create tag: v0.5.0
  6. Push commit and tag to origin

Proceed? (yes/no)
```

**Do NOT run `cargo release` without explicit "yes" from the user.**

### Step 6: Execute Release

```bash
cargo release {major|minor|patch} --execute
```

Where `{major|minor|patch}` matches the user-confirmed bump level from Step 3.

Monitor the output. If `cargo release` fails, show the error and stop.

### Step 7: Post-Release Verification

Verify the tag was pushed:

```bash
git tag -l "vX.Y.Z"
git ls-remote --tags origin "refs/tags/vX.Y.Z"
```

Remind the user:

> "Release v0.5.0 tagged and pushed. GitHub Actions will now:
> - Build release binaries for all targets
> - Create the GitHub Release with artifacts
> - Update the Homebrew tap
>
> Monitor at: https://github.com/jackin-project/jackin/actions/workflows/release.yml"

## Error Recovery

If something goes wrong at any step:

- **Before `cargo release`:** Safe to fix and re-run `/release`. The skill re-validates everything.
- **During `cargo release`:** Check what was committed/tagged. If the tag was created but not pushed, you can push it manually: `git push origin vX.Y.Z`. If the version bump commit was created but not tagged, you may need to reset and retry.
- **After `cargo release`:** The release is done. If CI fails, check the GitHub Actions workflow.
