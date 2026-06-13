---
name: merge-pr
description: Runs the jackin' pre-merge gate, retires the roadmap item into docs, and squash-merges a pull request. Use when the operator runs /jackin-dev:merge-pr.
disable-model-invocation: true
---

# merge-pr

Sequence the jackin' pre-merge gates **fail-closed**, retire the roadmap item into docs, then squash-merge. Reuses the `.github/` agent rules (auto-loaded) and `PULL_REQUESTS.md` — it sequences them, does not restate them.

## When to use

- The operator runs `/jackin-dev:merge-pr [<PR>]` to merge a jackin' PR (defaults to the current branch's PR).

## When NOT to use

- Opening or iterating a PR → `/jackin-dev:create-pr` / `/jackin-dev:propose`.

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
3. **CI gate.** `gh pr checks <PR>`: if pending, **poll until green** (unless `--no-poll`); if any check fails, STOP. Trust CI — no local fmt/clippy/test re-run.
4. **Roadmap retirement.** Does this PR ship the **last** piece of its roadmap item?
   - **Yes** → run the 7-step retire-into-docs: move remaining operator detail to `guides/`/`commands/`, design detail to `reference/`, delete the roadmap `.mdx`, replace with a Completed bullet in `roadmap/index.mdx`, repoint inbound links, run sidebar + overview audits, run the `bun` docs gate. Commit `docs:`; push.
   - **Partial** → set `**Status**: Partially implemented` with remaining phases named; sidebar/overview audits. Commit; push.
5. **Metadata reconcile.** Title/body match the final diff? Squash writes the title verbatim into history. Fix via `gh pr edit` if stale; surface the change in your reply.
6. **Squash-merge.** Build a body file (prose summary, no checklists), append trailers via `cargo run -p jackin-pr-trailers -- --body-file`, then `gh pr merge <PR> --squash --body-file`. Confirm the title carries `(#N)`.
7. **Report.** Merged SHA and what retirement did.

## Common mistakes

- Carrying a prior session's "just merge everything" as authorization — it does not carry forward.
- Merging on pending/failing CI, or `--admin`-bypassing without explicit per-check opt-in.
- Leaving a `Status: Resolved` roadmap page instead of retiring it into docs.
- Squash title missing the `(#N)` suffix or the DCO / Co-authored-by trailers.

## Tooling

Bundles `jackin-pr-trailers` (exists). A `cargo xtask roadmap retire <slug>` / `roadmap audit` helper (to be added) can mechanize the retirement steps; until then, do them directly.
