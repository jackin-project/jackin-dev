---
name: jackin-goal-prompt
description: Distills a jackin❯ roadmap item — plus optional plan files — into a self-contained /goal prompt capped at 4000 characters.
argument-hint: "<slug> [--plan <path|glob>]"
disable-model-invocation: true
---

# jackin-goal-prompt

Distill a roadmap item (and any plan files the operator hands you) into a single **self-contained** `/goal` prompt — one an executor agent with zero prior context can run to completion. The prompt is the product; its tightness decides whether the goal succeeds.

Capped at **4000 characters, hard.** Structure borrowed from `/improve`'s plans: self-contained, verification-gated, hard-boundaried. The skeleton that carries execution — objective, scope, verify gates, done criteria, STOP conditions — stays; everything the executor can read from files (code excerpts) or inherits from jackin❯'s auto-loaded rules (`AGENTS.md`, `COMMITS.md`, `ENGINEERING.md`) is pointed at, never restated.

Distinct from `jackin-research`: that authors a *dossier brief* (a multi-page `prompt.mdx` file run via `/goal Follow`). This produces a compact *inline* prompt run via `/goal Implement` or `/goal Run`.

## When to use

- Operator asks for a goal prompt / wants to kick off `/goal` on a roadmap item.
- Operator has a roadmap item plus `/improve` plan files and wants them fused into one runnable prompt.

## When NOT to use

- Authoring a research dossier brief → `jackin-research`.
- Designing the roadmap item itself → `jackin-brainstorm`.
- Opening the PR → `jackin-create-pr` / `jackin-propose`.

## Arguments

- `<slug>` — the roadmap item (slug, path, or the `.mdx`). Always required.
- `--plan <path|glob>` — plan markdown to fold in (a folder, a file, or a glob). Usually `/improve` output.

## Process

1. **Gather the change.** Read the roadmap `.mdx`. Anchor on `title:` and `**Status**:` only — real items drift from the `Problem/Why/Design/Tasks` scaffold, so tolerate any heading set. If `--plan` is given, read every plan file and lift its scope, steps, verify commands, done criteria, and STOP conditions.

   *Done when* you can state, in two sentences, what this change builds and where its code lives.

2. **Read the repo's real verification gates.** The verify commands come from the repo, never invented. Pull them from `CONTRIBUTING.md` (the merge-readiness block), `.github/workflows/ci.yml`, and `mise.toml` — for jackin❯ that is `cargo fmt --check`, `cargo clippy --all-targets --all-features -- -D warnings`, `cargo nextest run --all-features`, the `e2e` profile, plus the `bun` docs gate when `docs/` is touched.

   *Done when* every gate you cite is a command the repo actually runs, with its expected result.

3. **Compose the prompt** — the compressed `/improve` skeleton:
   - **Objective** — one or two lines: what to build, pulled from the roadmap/plan.
   - **Scope** — in-scope paths; out-of-scope boundaries (what looks related but must not be touched).
   - **Verify gates** — the commands from step 2, each with its expected result.
   - **Done criteria** — machine-checkable, one line each (a command + result, or a greppable absence), never prose like "works correctly."
   - **STOP conditions** — drift vs the plan, a scope boundary that won't hold, a gate that won't go green honestly.
   - **Conventions** — one line pointing at jackin❯'s auto-loaded rule files; do not copy their content.

4. **Enforce the 4000-character cap.** Write the prompt to a temp file and `wc -c` it. Over the cap, compress in this priority order until it fits: drop code excerpts (name the file + the contract, not the code), collapse conventions to a pointer, tighten the objective, cut the "why" narrative, merge done criteria into the verify gates. Re-count. The cap is hard — never emit over 4000.

   *Done when* `wc -c` reports ≤ 4000 and the prompt still carries objective + scope + verify gates + done criteria + STOP conditions.

5. **Emit + report.** Print the prompt in a single code block, ready to paste after `/goal Implement <slug>` or `/goal Run`. State the character count. Name what you compressed or dropped to fit, so the operator can ask for it back inline if they'd rather raise the cap.

## Common mistakes

- Exceeding 4000 characters — the one hard constraint.
- Inventing verify commands instead of reading the repo's real ones — a gate the repo doesn't run is noise.
- Restating jackin❯'s auto-loaded rules (`AGENTS.md`, `COMMITS.md`, `ENGINEERING.md`) — the executor auto-loads them; point, don't copy.
- Inlining code excerpts — the executor reads the files; name the file and the contract it must satisfy.
- Producing a prompt that leans on chat context ("as discussed", "the approach we chose") — not self-contained, breaks on a fresh executor.
- Ignoring `--plan` files when given — they carry the scope and done criteria; folding them in is the point.
- Dropping the STOP conditions — they are the executor's only honest escape hatch.

## Tooling

No xtask owns this. The skill orchestrates reads (the roadmap `.mdx`, the plan files, the repo's verify commands) and emits a single ≤ 4000-char string. The structure borrows from `/improve`'s plan anatomy (self-contained, verification-gated, hard-boundaried) and the `/goal Run` brief shape jackin❯ already uses for gap-closure.
