---
name: jackin-refresh-pr
description: Reconciles an open jackin❯ pull request's title and body against the current diff — regenerates the Verify-locally block selection and rewrites prose that has drifted.
argument-hint: "[PR]"
disable-model-invocation: true
---

# jackin-refresh-pr

Reconcile an open PR's title and body against the current diff, so the body describes what the branch **actually ships now** — not what it shipped when it was opened. Run when the body has drifted: more commits landed, scope grew or shifted, the title still reads `docs:` but the PR now ships a feature.

Shares `cargo xtask pr body` with `jackin-create-pr` — the binary owns the Verify-locally block selection; the agent owns the prose. This is the deliberate, on-demand version of the reconciliation `jackin-merge-pr` runs at its step 5.

## When to use

- Operator asks to refresh / update the PR body or title ("the body is out of date", "refresh the PR").
- Body has become actively misleading for a reviewer landing on it right now.

## When NOT to use

- Opening a new PR → `jackin-create-pr`.
- At merge-readiness → `jackin-merge-pr` step 5 reconciles as the final gate; don't double-refresh.
- After every iteration commit — refresh is operator-triggered, not commit-triggered (`.github/AGENTS.md` PR-body refresh policy). Auto-refreshing churns the body and wastes attention.

## Arguments

- `PR` — PR number (defaults to the current branch's PR).

## Process

1. **Resolve the PR.** `gh pr view <PR> --json number,title,body,headRefName,baseRefName`. Hold the current title and body.
2. **Gather the fresh shape.** `git fetch origin <baseRefName>`, then `cargo xtask pr body --base origin/<baseRefName> > /tmp/pr-body.fresh.md`. Read the **change digest on stderr** (the rust/docs/capsule/schema categories + file list) — that is what the change *is* now. Read the **fresh skeleton on stdout** — its set of `### ` blocks under `## Verify locally` is the selection the current diff requires.
3. **Reconcile the Verify-locally blocks.** Diff the fresh skeleton's block set against the live body's:
   - In the fresh selection but missing from the body → add it (structure from the template, filled for this PR).
   - In the body but dropped by the fresh selection → remove it.
   - In both → **keep the operator's fill verbatim**. The Checkout block carries the real PR number; Rust tests carry the scoped filter; User smoke carries PR-specific steps. Never overwrite a kept block with the placeholder.

   *Done when* every `### ` block under `## Verify locally` matches the binary's selection for the current diff, and every kept block retains its authored content.
4. **Reconcile the prose.** Read `gh pr diff <PR>` and `git log origin/<baseRefName>..HEAD --oneline`. For each prose section the body carries (Summary, What ships, Behavior changes, What this addresses, Hard rule, Not included, Migration notes):
   - Still matches what shipped → leave untouched. Refresh is anti-churn; do not rewrite what is right.
   - Drifted → rewrite to match the current diff. Section no longer earned → drop it. New outcome shipped with no section → add it.

   *Done when* every prose section reflects the current diff, and no section restates the diff file-by-file.
5. **Reconcile the title.** Does the Conventional Commits subject still describe the shipped scope? If the PR grew — a `fix(typo)` that now ships a feature, a `docs:` that now contains code — update type and scope via `gh pr edit <PR> --title`. Pause for confirmation only if the scope shift is meaningful enough that the operator might not have noticed (`.github/AGENTS.md` title/description rule).
6. **Write the body.** Build the reconciled body in a temp file — kept prose, rewritten prose, reconciled blocks — then `gh pr edit <PR> --body-file /tmp/pr-body.md`. Never `--body "..."`; the body-construction rule in `.github/AGENTS.md` mandates `--body-file` to survive code fences and `$`.
7. **Verify the render.** `gh pr view <PR> --json body -q .body` — confirm no stray `` \` `` or `\$` from hand-editing. Fix with `--body-file` if broken.
8. **Report.** Name what moved: blocks added/dropped, prose sections rewritten, the title change (old → new) and why. Do not ask permission to refresh — the operator asked. Do surface a scope-shifting title change before it sticks if the operator might disagree.

## Common mistakes

- Regenerating the body from the skeleton and **losing the operator's prose** — the skeleton carries placeholders; refresh preserves authored content and only rewrites what drifted.
- Rewriting prose that was already accurate — churns the diff and wastes attention.
- Hand-selecting Verify-locally blocks instead of `cargo xtask pr body` — the binary is the selection oracle.
- Using `gh pr edit --body "..."` instead of `--body-file` — breaks on code fences and `$`.
- Forgetting the render check — a broken `` \` `` or `\$` ships silently.
- Refreshing after every commit — violates the operator-triggered policy.

## Tooling

`cargo xtask pr body --base origin/<base>` (jackin❯ repo) emits the change digest to stderr and the Verify-locally-block-filtered skeleton to stdout. Shared with `jackin-create-pr`. The binary owns block selection; the agent owns prose reconciliation. No xtask subcommand refreshes a PR — this skill orchestrates `pr body` (selection) + `gh pr edit` (write).
