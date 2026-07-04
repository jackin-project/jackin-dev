---
name: jackin-release
description: Cuts a jackin❯ release — runs the readiness gates, writes the changelog, recommends a version, then executes cargo release on explicit operator confirmation.
argument-hint: "[major|minor|patch|version]"
disable-model-invocation: true
---

# jackin-release

**Cut** a release. The only irreversible skill in the suite — `cargo release --execute` bumps the version, rewrites the changelog header, tags, and pushes. Hard-gated on one explicit operator "yes"; nothing releases on assumption.

Orchestrates `jackin-release-check` (the gates) and `jackin-release-notes` (the changelog), then recommends a version from the changelog shape and cuts.

## When to use

- The operator asks to cut / ship / release a version.

## When NOT to use

- Just checking readiness → `jackin-release-check`.
- Just updating the changelog → `jackin-release-notes`.

## Prerequisites

`cargo-release` installed; `gh` authenticated; `release.toml` with `pre-release-replacements` for `CHANGELOG.md`; `CHANGELOG.md` with the `<!-- next-header -->` marker; `.github/workflows/ci.yml`.

## Process

1. **Gates.** Run `jackin-release-check`. A red gate **stops** — show the report and name what to fix. Warns surface; ask whether to continue.

   *Done when* every gate is green, or the operator accepted the warns.

2. **Changelog.** Run `jackin-release-notes`. Present the `[Unreleased]` section; let the operator re-classify or reword.

   *Done when* the operator approves the changelog.

3. **Recommend a version.** Read the approved `[Unreleased]` buckets: any **Removed** or "breaking" entry → **major**; any **Added** → **minor**; only **Fixed** / **Changed** / **Security** / **Deprecated** → **patch**. Read the current version (`grep '^version' Cargo.toml`), compute the next, present the recommendation, wait.

   *Done when* the operator confirms the bump level.

4. **Commit the changelog.** If `CHANGELOG.md` has uncommitted edits, `git add CHANGELOG.md && git commit -m "docs: update changelog for vX.Y.Z"`.

5. **Final confirm.** Show the summary — version, bump level, changelog shape, and the exact `cargo release <level> --execute` about to run (it bumps `Cargo.toml`, renames `[Unreleased]` to `[X.Y.Z] - <date>` via `release.toml`, tags, and pushes). **Do not run it without an explicit "yes."**

   *Done when* the operator says "yes."

6. **Cut.** `cargo release <level> --execute`. On failure, show the error and stop.

7. **Verify.** `git tag -l "vX.Y.Z"` and `git ls-remote --tags origin "refs/tags/vX.Y.Z"`. Point the operator at the release workflow that now builds the artifacts.

## Error recovery

- Before `cargo release`: safe to fix and re-run — this skill re-validates everything.
- During `cargo release`: check what committed or tagged. Tag created but unpushed → `git push origin vX.Y.Z`. Version commit without tag → reset and retry.
- After: the release is done; a CI failure is a workflow issue, not a rollback.

## Common mistakes

- Running `cargo release` without the explicit "yes" — the one hard gate.
- Letting a red readiness gate through to the cut.
- Hand-editing the `[X.Y.Z]` version heading — `cargo-release` rewrites it via `release.toml` `pre-release-replacements`; double-editing corrupts the release commit.
