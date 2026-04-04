---
name: release-check
description: Use when preparing for a release, verifying release readiness, or running pre-release checks on the jackin project
---

# Release Check

Pre-release validation for the jackin project. Runs a series of checks and produces a readiness report.

## When to Use

- Before running `/release`
- When you want to verify the project is ready to release
- After fixing issues found in a previous release check

## Process

Run each check in order. Collect results into a readiness report.

### Check 1: CI Status

Verify the latest CI run on `main` is green:

```bash
gh run list --workflow=ci.yml --branch=main --limit=1 --json status,conclusion --jq '.[0]'
```

If `conclusion` is not `success`, report **FAIL** and stop.

Also check path-specific workflows if their paths changed since the last tag:

```bash
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "")

# Check if docker/construct/** changed since last tag
if [ -n "$LAST_TAG" ] && [ -n "$(git diff --name-only "$LAST_TAG"..HEAD -- docker/construct/)" ]; then
  gh run list --workflow=construct.yml --branch=main --limit=1 --json status,conclusion --jq '.[0]'
fi

# Check if docs/** changed since last tag
if [ -n "$LAST_TAG" ] && [ -n "$(git diff --name-only "$LAST_TAG"..HEAD -- docs/)" ]; then
  gh run list --workflow=docs.yml --branch=main --limit=1 --json status,conclusion --jq '.[0]'
fi
```

Report result as **PASS**, **FAIL**, or **SKIP** (if no changes in those paths).

### Check 2: Local Tests

```bash
cargo test --locked
```

If any test fails, report **FAIL** and stop.

### Check 3: Clippy

```bash
cargo clippy -- -D warnings
```

If clippy reports any warnings-as-errors, report **FAIL** and stop.

### Check 4: Format Check

```bash
cargo fmt --check
```

If formatting violations found, report **FAIL** and stop.

### Check 5: Direct Commit Warning

Find commits since the last tag that are not merge commits from PRs:

```bash
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "")
if [ -n "$LAST_TAG" ]; then
  git log "$LAST_TAG"..HEAD --oneline --no-merges
fi
```

For each commit, check if it was part of a PR:

```bash
gh pr list --state merged --search "<commit-sha>" --json number --jq '.[0].number'
```

If there are commits not associated with any PR, report **WARN** with the list. Do not block.

### Check 6: Doc Link Validation

Check for broken internal links in `docs/src/content/docs/`:

- Read markdown files and extract internal links (relative paths, `href` attributes)
- Verify each linked file exists
- For external URLs, verify they return HTTP 200 (skip rate-limited domains)

Report **PASS** or **WARN** with list of broken links.

### Check 7: TODO Freshness

Read `TODO.md` and all files in `todo/`:

- Check if any items reference work that has already been completed (look for matching commits or closed PRs)
- Check if any items are stale (no related activity in recent commits)

Report **PASS** or **WARN** with findings.

### Check 8: Security Exceptions

Read `SECURITY_EXCEPTIONS.md` and present its contents to the user.

Ask: **"Are these security exceptions still current? (yes/no)"**

If the user says no, report **REVIEW** — the user needs to update the file before releasing.

If the user says yes, report **PASS**.

### Check 9: Docker Build Status

Check if Docker-related files changed since the last tag:

```bash
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "")
CONSTRUCT_CHANGED=$(git diff --name-only "$LAST_TAG"..HEAD -- docker/construct/ 2>/dev/null)
RUNTIME_CHANGED=$(git diff --name-only "$LAST_TAG"..HEAD -- docker/runtime/ 2>/dev/null)
```

If changed, verify the `construct.yml` workflow passed (already checked in Check 1). Report **PASS** or **SKIP**.

## Output

Present a structured readiness report:

```
Release Readiness Report
========================
✓ CI: all workflows green
✓ Local tests: N passed, 0 failed
✓ Clippy: no warnings
✓ Format: clean
⚠ Direct commits: N commits since vX.Y.Z not from PRs
  - <sha> <message>
✓ Doc links: all valid
✓ TODOs: up to date
? Security exceptions: review required (N items)
✓ Docker: builds pass (or: no changes, skipped)

Result: PASS | REVIEW NEEDED | FAIL
```

## Blocking vs Non-blocking

| Check | Failure behavior |
|---|---|
| CI status | **BLOCK** — cannot release with red CI |
| Local tests | **BLOCK** — cannot release with failing tests |
| Clippy | **BLOCK** — cannot release with clippy errors |
| Format | **BLOCK** — cannot release with fmt violations |
| Direct commits | **WARN** — show list, do not block |
| Doc links | **WARN** — show broken links, do not block |
| TODO freshness | **WARN** — show stale items, do not block |
| Security exceptions | **REVIEW** — ask user, block only if user says "no" |
| Docker builds | **BLOCK** if changed and CI failed; **SKIP** if unchanged |
