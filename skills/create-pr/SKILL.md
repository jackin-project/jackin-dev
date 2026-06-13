---
name: create-pr
description: Opens a pull request for a small jackin' change with the correct body shape and auto-selected verify-locally blocks. Use when the operator runs /jackin-dev:create-pr.
disable-model-invocation: true
---

# create-pr

Open a PR for a **small fix** in the jackin' repo — typo, dependency bump, one-line bugfix, doc tweak — that does not need a roadmap item. Also the shared PR-mechanics path other skills reuse. Commits inline; there is no separate commit skill.

The jackin' rule files auto-load — `PULL_REQUESTS.md`, `.github/` agent rules, `BRANCHING.md`, `COMMITS.md`. This skill sequences them; it does not restate them. Read them for the full rules.

## When to use

- The operator runs `/jackin-dev:create-pr`.
- The change is small and self-contained (no design, no roadmap item).

## When NOT to use

- A feature or idea worth a roadmap item → use `/jackin-dev:propose`.
- Implementing a finalized roadmap item → use `/goal Implement <slug>.md`.

## Arguments

- `--branch <name>` — explicit branch name.
- `--auto-branch` — pick the branch name yourself, no confirmation.
- `--title <msg>` — commit + PR title (else derive from the diff, Conventional Commits).

## Process

1. **Branch.** Never commit to `main`. If on `main`, create a `fix/` / `chore/` / `docs/` / `refactor/`-prefixed branch named from the change. Suggest and confirm unless `--auto-branch` or `--branch` was given.
2. **Commit.** If there are uncommitted changes, commit them inline per `COMMITS.md` — Conventional Commits subject, DCO sign-off (`git commit -s`). If already committed, skip. Then `git push`.
3. **Build the body.** Start from `.github/PULL_REQUEST_TEMPLATE.md` in the repo (read it at runtime — never embed a copy). Fill the prose sections. Auto-select the Verify-locally blocks from the changed paths:
   - Checkout + isolation env vars (`JACKIN_CONFIG_DIR`, `JACKIN_HOME_DIR`) — **always**.
   - Rust tests — when Rust sources changed.
   - Docs checks + Documentation walk — when `docs/**` changed.
   - jackin-capsule smoke + `--capsule` on the Checkout prepare — when `crates/jackin-capsule/` changed (the `--capsule` step must come before any `jackin console`/`load`).
   - Schema migration smoke — when a versioned schema changed.
4. **Create.** Write the body to a temp file with a single-quoted `<<'EOF'` heredoc (never escape backticks or `$`), then `gh pr create --body-file`.
5. **Verify render.** `gh pr view <PR> --json body -q .body`. If you see `\`` or `\$`, the body is broken — fix with `gh pr edit --body-file`.
6. **Reply.** Share the PR URL and repeat the Verify-locally commands in your final message.

## Common mistakes

- Committing to `main` instead of a branch.
- Embedding the PR template instead of reading `.github/PULL_REQUEST_TEMPLATE.md` at runtime.
- Escaping backticks / `$` inside the heredoc, producing broken rendered Markdown.
- Putting a `jackin console`/`load` step before the `--capsule` prepare for a capsule PR.
- Linking deployed docs URLs in the body (they break post-merge) — refer to docs by name.

## Tooling

A `cargo xtask pr body` helper (to be added in the jackin' repo) can mechanize the template read + verify-block selection. Until it exists, do those steps directly.
