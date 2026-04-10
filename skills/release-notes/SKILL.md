---
name: release-notes
description: Use when generating or updating the changelog, preparing release notes, or populating the Unreleased section of CHANGELOG.md for the jackin project
---

# Release Notes

Generates or updates the `[Unreleased]` section of `CHANGELOG.md` from merged PRs since the last release tag.

## When to Use

- Before a release, to populate the changelog
- When you want to preview what would go into the next release
- When asked to update or regenerate the changelog

## Process

### Step 1: Detect Last Tag

```bash
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "")
```

If no tags exist, this is the first release. Use the initial commit as the starting point:

```bash
FIRST_COMMIT=$(git rev-list --max-parents=0 HEAD)
```

### Step 2: Get the Tag Date

```bash
TAG_DATE=$(git log -1 --format=%aI "$LAST_TAG" 2>/dev/null || echo "")
```

### Step 3: Gather Merged PRs

```bash
gh pr list --state merged --base main --search "merged:>$TAG_DATE" --json number,title,labels,mergedAt --jq '.[] | {number, title, labels: [.labels[].name], mergedAt}'
```

### Step 4: Classify Each PR

Classify PRs into Keep a Changelog categories based on title prefix and labels:

| Title prefix | Label | Category |
|---|---|---|
| `feat:`, `feature:` | `feature`, `enhancement` | **Added** |
| `fix:` | `bug`, `bugfix` | **Fixed** |
| `security:` | `security` | **Security** |
| `refactor:`, `chore:`, `docs:`, `ci:`, `build:` | `refactor`, `chore`, `docs` | **Changed** |
| `deprecate:` | `deprecated` | **Deprecated** |
| `remove:` | `removed` | **Removed** |

If a PR doesn't match any prefix or label, place it in **Changed** as the default.

Strip the conventional commit prefix from the title when generating the entry. For example:
- `feat: add TUI launcher` becomes `Add TUI launcher`
- `fix: symlink escape in mounts` becomes `Fix symlink escape in mounts`

Capitalize the first letter of each entry.

### Step 5: Find Ungrouped Commits

Find commits since the last tag that are not associated with any merged PR:

```bash
git log "$LAST_TAG"..HEAD --oneline --no-merges
```

For each commit, check if it was part of a PR:

```bash
gh pr list --state merged --search "<commit-sha>" --json number --jq '.[0].number'
```

Commits not associated with a PR go into the **Ungrouped commits** section.

### Step 6: Format the Changelog Section

Format as Keep a Changelog with PR links:

```markdown
## [Unreleased]

### Added
- Add TUI launcher for interactive agent selection ([#12](https://github.com/jackin-project/jackin/pull/12))

### Fixed
- Fix symlink escape in container mounts ([#15](https://github.com/jackin-project/jackin/pull/15))

### Changed
- Extract testing instructions into TESTING.md ([#18](https://github.com/jackin-project/jackin/pull/18))
```

Only include sections that have entries. Do not include empty sections.

If there are ungrouped commits, add:

```markdown
### Ungrouped commits (not from PRs)
- `205875a` docs: fix resolve_agent_source references
- `efa41b8` docs: split TODO into individual files
```

### Step 7: Check for Existing Unreleased Section

Read `CHANGELOG.md` and check if an `[Unreleased]` section already has content.

If it does, ask the user:

> "CHANGELOG.md already has entries in the [Unreleased] section. Do you want to:
> A) Regenerate from scratch (replace existing entries)
> B) Keep existing entries and add any missing PRs"

### Step 8: Write to CHANGELOG.md

If `CHANGELOG.md` does not exist, create it with the standard header:

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

<!-- next-header -->

## [Unreleased]

[generated entries here]
```

If `CHANGELOG.md` exists, replace the `## [Unreleased]` section content (everything between `## [Unreleased]` and the next `## [` heading or end of file).

### Step 9: Present for Review

Show the complete `[Unreleased]` section to the user. Allow them to:

- Request edits ("move PR #42 to Added", "reword this entry", "remove this commit")
- Approve as-is

Do not proceed until the user approves the changelog content.

## Idempotency

This skill is safe to run multiple times:
- It always checks the current state of `CHANGELOG.md` before writing
- It asks before overwriting existing entries
- PR data is fetched fresh from GitHub each time
