---
name: propose
description: Opens a new jackin' feature or idea as a roadmap item draft plus an early pull request, without writing any code. Use when the operator runs /jackin-dev:propose.
argument-hint: "<idea> [--branch <name>|--auto-branch] [--no-pr] [--research]"
disable-model-invocation: true
---

# propose

Turn a feature, improvement, or problem into a tracked **roadmap item** and an early PR. This skill **never writes code** — it collects everything for the roadmap item, opens the PR, and stops. Implementation is always a separate later step (`/goal Implement <slug>.md`).

jackin' rule files auto-load — `BRANCHING.md`, `COMMITS.md`, `PULL_REQUESTS.md`, `docs/` rules, `TODO.md` (roadmap conventions). This skill sequences them.

## When to use

- Operator runs `/jackin-dev:propose <idea or problem>`.
- Feature/idea worth a roadmap item.

## When NOT to use

- Small fix (typo, dep bump, one-liner) → `/jackin-dev:create-pr`.
- Designing an existing item → `/jackin-dev:brainstorm`. Implementing one → `/goal Implement <slug>.md`.

## Arguments

- `--branch <name>` / `--auto-branch` — explicit / self-chosen branch name.
- `--no-pr` — branch + roadmap draft only, no PR (rare; default opens a PR).
- `--research` — run `deep-research` first and fold a short summary into the draft.

## Process

1. **Branch.** Never commit to `main`. Derive a `feature/<slug>` name (or `fix/` / `refactor/` / `chore/` per change type) from the idea; suggest and confirm unless `--auto-branch` / `--branch` given.
2. **(optional) Research.** On `--research`, run the built-in `deep-research` skill and summarize findings into the draft. For large research, prefer `/jackin-dev:research`.
3. **Scaffold the roadmap item.** Run `cargo xtask change new <slug> --group <group>` — it creates `docs/content/docs/roadmap/<slug>.mdx` (`**Status**: Open`, `## Problem`, `## Why It Matters`, `## Design`, `## Tasks`, `## Related Files`) and registers it in the chosen group's Fumadocs `meta.json`. Pick the `--group` from the existing roadmap groups. Then fill Problem / Why It Matters from the idea text, and run `cargo xtask roadmap audit`.
4. **Commit + push.** `docs:` type, DCO `-s`, then `git push`.
5. **Open the PR.** Unless `--no-pr`, build and open the PR with `create-pr` mechanics — Summary = the idea, What ships = "roadmap item draft for `<slug>`", Verify = docs render.
6. **Stop.** Point the operator at the next step: `/jackin-dev:brainstorm <slug>` to fill `## Design`. Do not implement.

## Common mistakes

- Writing code or filling `## Design` / `## Tasks` — those belong to `brainstorm` / `plan` / `goal`, never `propose`.
- Skipping the sidebar entry or audit, leaving the item unreachable.
- Forgetting the early PR (it is the default — only `--no-pr` skips it).

## Tooling

`cargo xtask change new <slug> --group <group>` scaffolds the `.mdx` + registers the Fumadocs `meta.json` entry; `cargo xtask roadmap audit` validates the sidebar.
