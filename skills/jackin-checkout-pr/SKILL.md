---
name: jackin-checkout-pr
description: Switches the current jackin❯ repo onto a pull request's branch via gh pr checkout, guarding the working tree first.
argument-hint: "<PR number | URL | branch>"
disable-model-invocation: true
---

# jackin-checkout-pr

Switch the current working repo onto a PR's branch, so you can work on it from where you are — not the isolated verification bundle. Accepts a PR number, a PR URL, or a branch name. Always checks out via `gh pr checkout` (the GitHub CLI); never raw `git checkout`.

Distinct from `jackin-dev pr sync <N>` (the PR template's verify-locally flow): that builds a separate isolated bundle under `~/Projects/jackin-project/test/pr-<N>/` for running the verify blocks. This skill switches the repo you are standing in.

## When to use

- Operator asks to check out / switch to / pull a PR locally ("checkout PR 714", "switch to the docs-plans branch", pastes a PR URL).
- Operator is on `main` (or another branch) and wants to land on the PR's branch to continue work.

## When NOT to use

- Running a PR's verify-locally blocks → use `jackin-dev pr sync <PR>` from the PR template (isolated bundle, not a working-repo switch).
- Creating a new PR → `jackin-create-pr`.

## Arguments

- `<PR number | URL | branch>` — the identifier. A number or URL passes straight to `gh pr checkout`; a branch name is resolved to its PR first.

## Process

1. **Normalize the identifier.**
   - Number (`714`) or URL (`.../pull/714`) → pass to `gh pr checkout` directly (gh accepts both).
   - Branch name (`docs/plans-foo`) → resolve: `gh pr list --head <branch> --state all --json number -q '.[0].number'`. No PR matches → STOP and surface; do not fall back to raw `git checkout` (the operator mandated the GitHub CLI).

   *Done when* you hold a PR number `gh pr checkout` accepts.

2. **Read the state you are leaving.** `git branch --show-current` (note it — it is the way back). Then `git status --porcelain`. If anything is dirty → STOP, list the dirty paths, and ask the operator to commit, stash, or discard first. Never auto-stash — that is a silent state write the host-write rule forbids.

   *Done when* the working tree is clean and the branch being left is recorded.

3. **Confirm the PR is open.** `gh pr view <N> --json state,headRefName,headRefOid`. If `MERGED` or `CLOSED` → warn (the branch may be deleted or stale) and confirm before proceeding.

   *Done when* the PR is open, or the operator confirmed a non-open one.

4. **Switch.** `gh pr checkout <N>`.

   *Done when* the command succeeds.

5. **Verify the landing.** `git branch --show-current` matches `headRefName` from step 3.

   *Done when* the current branch is the PR's head branch.

6. **Report.** Switched from `<old>` to `<headRefName>` (PR #N, `<state>`). The way back: `git switch <old>` (or `git switch -`). Restate the warning if the PR was not open.

## Common mistakes

- Switching with a dirty working tree — uncommitted changes follow you or conflict.
- Auto-stashing to "help" — silent state mutation, violates the host-write rule.
- Using `git checkout <branch>` instead of `gh pr checkout` — the operator mandated the GitHub CLI.
- Not recording the branch being left — the operator cannot easily return.
- Silently landing on a `MERGED` / `CLOSED` PR without warning.
- For a branch-name input, silently falling back to `git fetch` + `git checkout` when no PR matches — surface instead.
- Confusing this with `jackin-dev pr sync` — that builds the isolated verification bundle; this switches the working repo.
