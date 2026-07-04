---
name: jackin-checkout-pr
description: Switches the current jackin❯ repo onto a pull request's branch via gh pr checkout, guarding the working tree first.
argument-hint: "<PR number | URL | branch>"
disable-model-invocation: true
---

# jackin-checkout-pr

Switch the working repo onto a PR's branch via `gh pr checkout`. Accepts a number, URL, or branch name.

## Process

1. **Resolve to a PR number.** Number or URL → pass to `gh pr checkout` directly. Branch name → `gh pr list --head <branch> --state all --json number -q '.[0].number'`; no match → STOP, do not fall back to raw `git checkout`.

2. **Guard the working tree.** `git status --porcelain` — if dirty, STOP and ask the operator to commit, stash, or discard first. Never auto-stash. Note `git branch --show-current` (the way back).

3. **Check the PR is open.** `gh pr view <N> --json state,headRefName`. `MERGED` or `CLOSED` → warn (the branch may be stale) and confirm before proceeding.

4. **Switch.** `gh pr checkout <N>`. Verify `git branch --show-current` matches `headRefName`.

5. **Report.** Switched from `<old>` to `<headRefName>` (PR #N). Way back: `git switch <old>`.
