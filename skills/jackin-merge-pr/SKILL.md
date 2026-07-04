---
name: jackin-merge-pr
description: Runs the jackin❯ pre-merge gate, retires the roadmap item into docs, and squash-merges a pull request.
argument-hint: "[PR] [--no-poll] [--admin <check>]"
disable-model-invocation: true
---

# jackin-merge-pr

Sequence the jackin❯ pre-merge gates **fail-closed**, retire the roadmap item into docs, then squash-merge. Reuses the `.github/` agent rules (auto-loaded) and `PULL_REQUESTS.md` — sequences them, never restates them.

## When to use

- Operator runs `jackin-merge-pr [<PR>]` to merge a jackin❯ PR (defaults to the current branch's PR).

## When NOT to use

- Opening or iterating a PR → `jackin-create-pr` / `jackin-propose`.

## Arguments

- `--no-poll` — do not wait on pending CI; stop and report instead.
- `--admin <check>` — authorize bypassing one named failing check (still needs the high-blast-radius confirm).

## Red flags — STOP

- No clear authorization for **this** PR and it is high-blast-radius → STOP, confirm first.
- Any required CI check failing → STOP (no `--admin` bypass without `--admin <check>` + confirm).
- Roadmap item not retired / not status-moved when the PR ships or advances it → STOP, fix first.

## Process

1. **Resolve PR.** Current branch's PR or the arg. Read `gh pr view` and `gh pr diff`.
2. **Blast-radius classify.** *High* if the diff touches a versioned schema, auth/security surface, `.github/workflows/**`, or needs a force-push / `--admin`. High → pause for one explicit confirm. Normal → proceed (invoking the skill is the go).
3. **CI gate.** `gh pr checks <PR>`: if pending, **poll until green** (unless `--no-poll`); if any check fails, STOP. Trust CI — which runs `cargo xtask schema-check` plus the test/clippy/fmt jobs — no local re-run.
4. **Roadmap retirement.** Does this PR ship the **last** piece of its roadmap item?
   - **Yes** → `[bin]` `cargo xtask roadmap retire <slug> --plan` prints the worklist: the page content, every inbound link (`grep <slug>`), and the `meta.json` entry. `[agent]` Do the judgment moves — operator detail → `guides/`/`commands/`, design detail → `reference/`, write the `## Completed` bullet in `roadmap/index.mdx`, repoint inbound links; commit `docs:`. `[bin]` `cargo xtask roadmap retire <slug> --apply` removes the `meta.json` entry, deletes the `.mdx`, runs the audit, and **fails if any inbound link still points at the dead slug**. Run the `bun` docs gate; push.
   - **Partial** → `cargo xtask roadmap retire <slug> --partial` sets `**Status**: Partially implemented` and keeps the page; name the remaining phases; commit; push.
5. **Metadata reconcile.** Title/body match the final diff? Squash writes the title verbatim into history. Fix via `gh pr edit` if stale; surface the change in your reply.
6. **Squash-merge.** Build a body file (prose summary, no checklists), append trailers via `cargo run -p jackin-pr-trailers -- --body-file`, then `gh pr merge <PR> --squash --body-file`. Confirm the title carries `(#N)`.
7. **Report.** Merged SHA and what retirement did.

## Common mistakes

- Carrying a prior session's "just merge everything" as authorization — it does not carry forward.
- Merging on pending/failing CI, or `--admin`-bypassing without explicit per-check opt-in.
- Leaving a `Status: Resolved` roadmap page instead of retiring it into docs.
- Squash title missing the `(#N)` suffix or the DCO / Co-authored-by trailers.

## Tooling

Bundles `jackin-pr-trailers`. `cargo xtask roadmap retire <slug> --plan` gives the agent the retirement worklist; `--apply` does the mechanical removal + audit and fails on a dangling link; `--partial` sets the Status. The content moves themselves are the agent's judgment. CI runs `cargo xtask schema-check` (the 5-artifact gate); this skill only reads its result.
