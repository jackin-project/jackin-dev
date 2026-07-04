---
name: jackin-release-notes
description: Classifies every PR merged since the last tag into Keep a Changelog categories and writes the [Unreleased] section of CHANGELOG.md.
argument-hint: "[context]"
disable-model-invocation: true
---

# jackin-release-notes

**Classify** every PR merged since the last release tag into a Keep a Changelog bucket, then write the `[Unreleased]` section of `CHANGELOG.md`. The bar is exhaustive — every merged PR accounted for, none dropped, none doubled.

## When to use

- Before a release, to populate the changelog.
- When the operator asks to preview what the next release would contain.

## When NOT to use

- Cutting the release itself → `jackin-release` (it runs this skill as its notes step).

## Process

1. **Find the boundary.** `LAST_TAG=$(git describe --tags --abbrev=0)`; if none, start from the initial commit (`git rev-list --max-parents=0 HEAD`). `TAG_DATE=$(git log -1 --format=%aI "$LAST_TAG")`.

   *Done when* you hold the boundary commit and its date.

2. **Gather merged PRs.** `gh pr list --state merged --base main --search "merged:>$TAG_DATE" --json number,title,labels,mergedAt`.

   *Done when* every PR merged after the tag is in the list.

3. **Classify each.** Sort by Conventional Commits prefix and label:

   | Prefix / label | Bucket |
   |---|---|
   | `feat:` / `feature` | **Added** |
   | `fix:` / `bug` | **Fixed** |
   | `security:` | **Security** |
   | `refactor:`, `chore:`, `docs:`, `ci:`, `build:` | **Changed** |
   | `deprecate:` | **Deprecated** |
   | `remove:` | **Removed** |
   | no match | **Changed** (default) |

   Strip the prefix from the title; capitalize the first word (`feat: add TUI launcher` → `Add TUI launcher`).

   *Done when* every PR sits in exactly one bucket.

4. **Catch ungrouped commits.** `git log <last-tag>..HEAD --oneline --no-merges`; for each, `gh pr list --state merged --search "<sha>"`. Commits with no PR land under **Ungrouped commits**.

   *Done when* every non-merge commit since the tag is matched to a PR or listed.

5. **Write `[Unreleased]`.** Keep a Changelog format: one section per non-empty bucket, each entry linking its PR (`[#N](…/pull/N)`). Replace the existing `[Unreleased]` block (between `## [Unreleased]` and the next `## [`) — ask before overwriting entries an operator already edited. Keep the `<!-- next-header -->` marker intact.

   *Done when* `CHANGELOG.md` carries every merged PR under the right bucket, empty buckets omitted, and the marker survives.

6. **Present for review.** Show the section; let the operator re-classify or reword. Do not proceed to a release until approved.

## Idempotency

Safe to re-run: reads `CHANGELOG.md` fresh each time, asks before overwriting existing entries, fetches PR data live from GitHub.

## Common mistakes

- Dropping a PR that has no conventional-commit prefix — it defaults to **Changed**, never omitted.
- Listing an empty bucket ("no **Removed** entries" is noise — drop the section).
- Forgetting the PR link in each entry.
- Overwriting an operator-edited `[Unreleased]` without asking.
