---
name: create-pr
description: Opens a pull request for a small jackin' change with the correct body shape and auto-selected verify-locally blocks. Use when the operator runs /jackin-dev:create-pr.
disable-model-invocation: true
---

# create-pr

Open a PR for a **small fix** in the jackin' repo — typo, dependency bump, one-line bugfix, doc tweak — that needs no roadmap item. Also the shared PR-mechanics path other skills reuse. Commits inline; no separate commit skill.

jackin' rule files auto-load — `PULL_REQUESTS.md`, `.github/` agent rules, `BRANCHING.md`, `COMMITS.md`. This skill sequences them, never restates them. Read them for full rules.

## When to use

- Operator runs `/jackin-dev:create-pr`.
- Small, self-contained change (no design, no roadmap item).

## When NOT to use

- Feature/idea worth a roadmap item → `/jackin-dev:propose`.
- Implementing a finalized roadmap item → `/goal Implement <slug>.md`.

## Arguments

- `--branch <name>` — explicit branch name.
- `--auto-branch` — pick the branch name yourself, no confirmation.
- `--title <msg>` — commit + PR title (else derive from the diff, Conventional Commits).

## Process

1. **Branch.** Never commit to `main`. If on `main`, create a `fix/` / `chore/` / `docs/` / `refactor/`-prefixed branch named from the change. Suggest and confirm unless `--auto-branch` or `--branch` given.
2. **Commit.** Uncommitted changes → commit inline per `COMMITS.md`: Conventional Commits subject, DCO sign-off (`git commit -s`). Already committed → skip. Then `git push`.
3. **Build the body.** `[bin]` Run `cargo xtask pr body --base origin/main`. It prints a classified change digest (rust / docs / capsule / schema + diffstat) and the PR template — read from `.github/PULL_REQUEST_TEMPLATE.md` at runtime — with the verify-locally blocks already selected and byte-exact, prose left as placeholders. Block selection: Checkout + isolation env (`JACKIN_CONFIG_DIR`, `JACKIN_HOME_DIR`) always; Rust tests when Rust changed; Docs checks + walk when `docs/**` changed; jackin-capsule smoke with `--capsule` *before* any `console`/`load` when `crates/jackin-capsule/` changed; schema migration smoke when a versioned schema changed. `[agent]` Fill the prose sections (Summary, What ships, Behavior changes) from the digest + diff.
4. **Create.** Write the body to a temp file with a single-quoted `<<'EOF'` heredoc (never escape backticks or `$`), then `gh pr create --body-file`.
5. **Verify render.** `gh pr view <PR> --json body -q .body`. If you see `\`` or `\$`, the body is broken — fix with `gh pr edit --body-file`.
6. **Reply.** Share the PR URL and repeat the Verify-locally commands in your final message.

## Common mistakes

- Committing to `main` instead of a branch.
- Embedding the PR template instead of reading `.github/PULL_REQUEST_TEMPLATE.md` at runtime.
- Escaping backticks / `$` inside the heredoc → broken rendered Markdown.
- A `jackin console`/`load` step before the `--capsule` prepare on a capsule PR.
- Deployed docs URLs in the body (break post-merge) — refer to docs by name.

## Tooling

`cargo xtask pr body` (jackin' repo) emits the change digest + the template skeleton with verify-locally blocks auto-selected from the diff. Shared with `propose`. The binary guarantees the mechanical parts (block selection, capsule ordering, exact command text, heredoc quoting); the agent writes the prose.
