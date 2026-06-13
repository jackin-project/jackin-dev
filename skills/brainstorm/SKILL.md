---
name: brainstorm
description: Turns a jackin' roadmap item's intent into concrete design decisions through freeform discussion, written into its Design section. Use when the operator runs /jackin-dev:brainstorm.
disable-model-invocation: true
---

# brainstorm

Turn an existing roadmap item's intent (and any gathered research) into concrete **design decisions**, through open back-and-forth with the operator, written incrementally into the item's `## Design` section.

Judgment task — describe and decide; do not script it, do not write code, do not break work into tasks (that is `plan`, deferred).

## When to use

- Operator runs `/jackin-dev:brainstorm <slug>` to design an open roadmap item.

## When NOT to use

- Creating the item → `/jackin-dev:propose`.
- Large external/codebase investigation → `/jackin-dev:research`.
- Implementing → `/goal Implement <slug>.md`.

## Arguments

- `--research` — run `/jackin-dev:research` first if the item looks thin.
- `--resume` — continue from the current `## Design` state.

## Process

1. **Load context.** Read `docs/content/docs/reference/roadmap/<slug>.mdx` — Problem, Why It Matters, Design-so-far, Related Files — plus any research already linked.
2. **Discuss freeform.** Open back-and-forth: surface alternatives, trade-offs, open questions. Pull in jackin' design principles and the relevant rule files (`ENGINEERING.md`, `HOST_AND_CONTAINER.md`, TUI/docs rules) as constraints. Prefer one question at a time; check in as each part settles.
3. **Write incrementally.** As each point settles, append or update the matching part of `## Design` in the `.mdx` — the decision plus a one-line *why*. `--resume` picks up from there.
4. **Hit an unknown?** Gather quickly (Explore/grep, or `deep-research`). Large investigation → spin a separate `/jackin-dev:research` dossier and link it from `## Design`. Then resume deciding.
5. **Converge.** When `## Design` covers the approach end to end, summarize and point at the implementation step (`/goal Implement <slug>.md`).
6. **Commit + push.** `docs:` per settled chunk (push after every commit).

## Common mistakes

- Writing code, or filling `## Tasks` — out of scope.
- Dumping a finished design without the back-and-forth, or asking many questions at once.
- Recording decisions only in chat instead of into `## Design` (it must be resumable).
- Restating rule-file content instead of referencing it by name.
