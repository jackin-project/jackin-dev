# jackin' skills proposal — working doc

Living document. We iterate here. Captures: how the current jackin' workflow reads from the repo's rule files, what Claude Code skill best-practices constrain, which skills are worth creating, and the open questions whose answers shape the final set.

These skills are destined for **this repository** (the `jackin-dev` plugin), alongside `release` / `release-check` / `release-notes`.

Status legend: 🟢 agreed · 🟡 proposed, needs input · ⚪️ parked

---

## 1. How the current workflow reads

The `jackin-project/jackin` repo is governed by a dense, well-factored rule graph (`AGENTS.md` slim index → topic files: `PULL_REQUESTS.md`, `BRANCHING.md`, `COMMITS.md`, `PRERELEASE.md`, `ENGINEERING.md`, `TESTING.md`, `HOST_AND_CONTAINER.md`, `RULES.md`, `PROJECT_STRUCTURE.md`, `TODO.md`, plus subtree `AGENTS.md` under `.github/`, `docs/`, `crates/`). The harness **auto-loads** `AGENTS.md` and the relevant subtree files whenever work happens in a directory. So the *static knowledge* is already in context for free.

What the rule files do **not** give is **procedural sequencing on demand** — a triggered, ordered checklist that walks a multi-step flow and fails closed. That gap is where skills earn their keep.

Repeatable flows extracted, each currently spread across several files:

| Flow | Rules it touches | Why it's painful by hand each time |
|---|---|---|
| **Open a PR** | `PULL_REQUESTS.md`, `.github/PULL_REQUEST_TEMPLATE.md`, `.github/` agent rules | 8-section body shape, optional-section drop logic, verify-locally block *selection* (Checkout always; Static/Rust/Schema/Docs/User-smoke/capsule conditional), isolation env vars, `--capsule` ordering, docs-only path, no-deployed-links, `--body-file` heredoc construction, repeat verify cmds in final reply |
| **Merge a PR** | `.github/` agent rules, `PULL_REQUESTS.md`, `BRANCHING.md` | per-PR auth gate, `gh pr checks` green, title/desc reconciliation vs diff, roadmap-freshness re-check, squash title `(#N)`, `jackin-pr-trailers` body, design-principle final gate |
| **Review a PR** | `PULL_REQUESTS.md` review section, `PRERELEASE.md`, TUI + design-principles docs | 4 cross-cutting gates (schema-migration 5 artifacts, accepted-exceptions catalog, design-principles, TUI design decisions) + solo-maintainer multi-agent review substitute |
| **Bump a versioned schema** | `PRERELEASE.md`, `PULL_REQUESTS.md` migration check | the "five artifacts under one bump" rule; easy to ship 3 of 5 and pass unit tests but break the migration-chain fixtures |
| **Smoke-test / walk verify-locally** | `TESTING.md`, template Verify-locally | `--debug` mandate + run-id triage, console-first, `the-architect` over `agent-smith`, capsule export ordering, isolation dirs |
| **Keep docs + roadmap fresh** | `PROJECT_STRUCTURE.md` cross-ref, `TODO.md` stale-docs, `PULL_REQUESTS.md` docs gate, `docs/` rules | code↔docs cross-ref table, three-audience split, roadmap status moves, roadmap retirement (7 steps), `bun` docs gate |
| **Branch / commit hygiene** | `BRANCHING.md`, `COMMITS.md` | stay-on-active-branch, prefix naming, rebase-not-merge, Conventional Commits, DCO `-s`, push-after-every-commit |
| **Record capsule fixtures** | `TESTING.md` | `cargo xtask pty-fixture <run.jsonl> <label> <out.bin>` from a `--debug` run |
| **Author/modify CI workflows** | `.github/` GitHub Actions rules | mise-only installs, job-level env scope, publish gate on `main`, PR/main parity, smoke push-only jobs |

Already covered by existing skills (do **not** rebuild): `release`, `release-check`, `release-notes` (this plugin); generic `commit`, `commit-push-pr` (commit-commands); `/code-review`, `/security-review`, `/verify`, `/run` (built-in).

---

## 2. Best-practice constraints (Anthropic skill-authoring guide)

These shape *how* each skill is written, not *whether*:

- **Naming:** gerund form, lowercase-hyphen, ≤64 chars. e.g. `opening-jackin-prs`.
- **Description:** third person, states **what + when**, packs trigger keywords, ≤1024 chars. This is the *only* thing pre-loaded — it decides whether the skill fires.
- **Body ≤500 lines.** Overflow → split into reference files, linked **one level deep** from SKILL.md.
- **Degrees of freedom:** fragile fixed sequences (schema bump, capsule ordering, squash/trailers) → **low freedom** (exact commands, "do not modify"). Judgment tasks (review) → **high freedom** (heuristics).
- **Scripts for deterministic checks** beat prose: a validator that *asserts* the 5 migration artifacts exist is more reliable than a checklist Claude eyeballs.
- **Don't restate the rule files.** They auto-load. Skills reference them by name ("see the schema rule in `PRERELEASE.md`") and add the *sequence* + *scripts*.
- **No time-sensitive info; consistent terminology; forward-slash paths.**

---

## 3. Design principle for this skill set

**Skills are the orchestration layer; the topic files stay the source of truth.** Each skill = a triggered checklist that (a) sequences steps scattered across topic files, (b) calls validator scripts where a step is mechanical, (c) names the governing rule file rather than copying it. When a rule changes, it changes in one place (the topic file); the skill keeps pointing at it. This keeps skills thin and rot-resistant — consistent with the jackin' repo's own "single source of truth / symlink, never copy" ethos.

Home: the **`jackin-dev` plugin** (this repo), so the skills version with the toolchain and install via the marketplace.

---

## 4. Candidate skills

Priority = daily friction removed × how error-prone the manual path is.

| # | Skill (gerund) | Trigger intent | Sequences / bundles | Freedom | Script? | Priority |
|---|---|---|---|---|---|---|
| S1 | `opening-jackin-prs` | "open a PR", "raise a PR for this" | body shape, verify-block selection, isolation env, capsule/docs-only branches, `--body-file` heredoc, repeat-verify reply | mixed | optional body-file builder | 🔥 highest |
| S2 | `merging-jackin-prs` | "merge it", "ship this PR" | auth gate, CI-green, title/desc reconcile, roadmap re-check, squash `(#N)`, `jackin-pr-trailers` | low | uses `jackin-pr-trailers` | 🔥 highest |
| S3 | `bumping-jackin-schema` | touch `AppConfig`/`WorkspaceConfig`/`RoleManifest`/`HooksConfig` | 5-artifact gate, one-bump-per-PR, fixture scaffold + re-bake, timeline entry | low | **yes** — `cargo xtask schema-check` | 🔥 high |
| S4 | `reviewing-jackin-prs` | "review this PR/diff" | 4 cross-cutting gates + multi-agent fan-out (solo-maintainer model) | high | reuses `schema-check` | high |
| S5 | `smoke-testing-jackin` | "verify this PR locally", "walk me through testing" | `--debug` mandate, run-id triage, console-first + `the-architect`, capsule export, isolation dirs | low-med | no | med |
| S6 | `syncing-jackin-docs` | "update docs", pre-merge docs gate | code↔docs cross-ref, three-audience split, roadmap status/retire, `bun` gate | med | optional cross-ref/audit script | med |
| S7 | `recording-capsule-fixtures` | "record a PTY fixture", "capture render-conformance" | `--debug` run → `cargo xtask pty-fixture` → `include_bytes!` wiring | low | no (xtask is the script) | med-low |
| S8 | `editing-jackin-ci` | touch `.github/workflows/**` | mise-only, job-level env, publish-gate-on-main, PR/main parity, smoke push-only jobs | med | optional parity linter | low-med |

Folded, not standalone (cover inside S1/S2 instead of separate skills): branch setup, commit hygiene — short, and the `commit-commands` plugin + `BRANCHING.md`/`COMMITS.md` already carry them. See open question Q4.

---

## 5. Full specs — S1–S4 (build targets)

Each skill is a directory under `skills/<name>/`. `SKILL.md` body stays ≤500 lines; overflow → linked reference files one level deep.

### S1 — `opening-jackin-prs`  🟡

- **Description:** "Opens a pull request for the jackin' repository with the correct body shape, verify-locally blocks, and isolation env vars. Use when the user asks to open, create, or raise a PR for jackin'. Handles the capsule, docs-only, and schema-migration variants and builds the body with a heredoc body-file so GitHub renders it correctly."
- **Freedom:** mixed — high for prose (Summary/What ships), low for the heredoc + isolation + capsule ordering.
- **SKILL.md flow:**
  1. **Pre-flight** — confirm on a feature branch (not `main`); if on `main`, propose a prefixed name and stop for confirmation (`BRANCHING.md`).
  2. **Classify the change** — from changed paths, decide variant(s): touches `crates/jackin-capsule/` → capsule; `docs/**` only → docs-only; serializes into the 3 versioned files → schema-migration (hand off to S3 first).
  3. **Select verify-locally blocks** — run the path→block matrix (reference file). Checkout + isolation env always; the rest conditional.
  4. **Draft body** — 8 sections in order, drop optional ones per template comments; no hard-wrap, no deployed-docs links, no file-by-file changelog.
  5. **Construct + create** — single-quoted `<<'EOF'` heredoc → `gh pr create --body-file`. Never escape backticks/`$`.
  6. **Verify render** — `gh pr view <PR> --json body -q .body`; if `\`` or `\$` present, fix via `gh pr edit --body-file`.
  7. **Reply** — share URL + repeat the verify-locally commands in the final message.
- **Reference files:** `verify-block-selection.md` (path → block matrix + capsule `--capsule` ordering rule), `body-shape.md` (8 sections + anti-patterns).
- **Scripts:** optional `build-body.sh` helper (writes heredoc to tmp, runs create, reads back render). Decide in follow-ups.
- **Earns its keep:** the path→block selection and the capsule-ordering invariant are prose-only in the rules; the skill makes them a deterministic walk + a render self-check.

### S2 — `merging-jackin-prs`  🟡

- **Description:** "Runs the jackin' pre-merge gate and squash-merges a PR. Confirms explicit per-PR merge authorization, verifies CI is green via `gh pr checks`, reconciles the PR title and description against the diff, re-checks roadmap and docs freshness, then squash-merges with a `(#N)` title and deduped Signed-off-by / Co-authored-by trailers. Use only when the user explicitly authorizes merging a specific jackin' PR."
- **Freedom:** low — exact commands, explicit STOP gates.
- **SKILL.md flow (fail-closed):**
  1. **Authorization gate** — require explicit "merge it / ship it" for *this* PR. Prior-session or sibling-PR authorization does NOT carry. Else stop and ask.
  2. **CI gate** — `gh pr checks <PR>`; any non-`pass` → stop. No `--admin` bypass without per-failure operator opt-in.
  3. **Freshness gate** — re-walk diff for roadmap/docs staleness; if stale, update + push before merging.
  4. **Metadata reconcile** — `gh pr view` + `gh pr diff`; fix stale title/body via `gh pr edit` (squash writes title verbatim into history). Surface the change in the reply.
  5. **Squash** — build body file (prose summary, no checklists), append trailers via `cargo run -p jackin-pr-trailers -- --body-file`, then `gh pr merge <PR> --squash --body-file`. Confirm title carries `(#N)`.
- **Bundles:** `jackin-pr-trailers`.

### S3 — `bumping-jackin-schema`  🟡  *(script-backed)*

- **Description:** "Guides a versioned-schema change to jackin' config, per-workspace files, or the role manifest. Bumps exactly one `CURRENT_*_VERSION`, adds the migration step, scaffolds the new fixture directory, re-bakes every existing `after.toml`, and adds the schema-versions timeline entry. Use when a change touches `AppConfig`, `WorkspaceConfig`, `RoleManifest`, `HooksConfig`, or any type whose serde form lives in those three files."
- **Freedom:** low — the five artifacts are a fixed sequence.
- **SKILL.md flow:** identify file-kind → confirm single next-version bump (no gap/skip; rebase if `main` advanced) → add migration step in the right registry → scaffold `tests/fixtures/migrations/<kind>/from-<pred>/{meta,before,after}.toml` → re-bake all existing `after.toml` → add `schema-versions.mdx` timeline entry → run the validator → run fixture tests.
- **Script (D2):** new `cargo xtask schema-check` (working name) — given the working tree / diff, asserts all five artifacts are present and consistent (version bumped, migration registered, fixture dir exists, every `from_version` re-baked, timeline entry added). Verbose, specific "missing X" errors. Wire the same subcommand into CI so PR-time and main-time enforce identically (PR/main parity rule). **This xtask is a prerequisite deliverable for S3.**

### S4 — `reviewing-jackin-prs`  🟡  *(thin orchestrator, D3)*

- **Description:** "Reviews a jackin' PR against the four cross-cutting gates — versioned-schema migration completeness, the accepted-exceptions catalog, the design principles, and the TUI design decisions — then runs the multi-agent review that substitutes for a second human reviewer. Use when asked to review a jackin' PR or diff."
- **Freedom:** high (judgment), but the 4 gates are mandatory checks.
- **SKILL.md flow:**
  1. **Gate 1 — schema migration:** if diff touches the 3 versioned files, run `cargo xtask schema-check` (shared with S3); flag missing artifacts.
  2. **Gate 2 — accepted-exceptions:** consult the open-review-findings catalog on demand; don't flag catalogued items.
  3. **Gate 3 — design principles:** read the design-principles page; flag any contradiction with the prescribed "Operator decision required" phrasing.
  4. **Gate 4 — TUI:** for console/capsule/TUI diffs, check the documented interaction cues (progress state, clickable affordances, footer hints, focus/scroll). Flag with the prescribed phrasing.
  5. **Delegate:** fan out `pr-review-toolkit` agents / `/code-review` for correctness, silent-failure, type-design; collect.
- **Earns its keep:** bundles the 4 repo-specific gates the generic reviewers don't know about, then reuses existing review machinery instead of duplicating it.

### S5–S8 — parked (specs on request)
Smoke-testing, docs/roadmap sync, capsule fixtures, CI authoring. To be specced after S1–S4 are agreed and once Q5/Q4 resolve.

---

## 6. Open questions

1. **Scope priority** — Which skills first? Current plan: S1–S4. ✅ (see decisions)
2. **Schema-migration script** — xtask vs shell. ✅ xtask (D2).
3. **Review skill shape** — thin orchestrator vs standalone. ✅ thin (D3).
4. **Branch/commit hygiene** — fold into S1, or a separate `starting-jackin-work` skill handling branch creation + naming + stay-on-active-branch at session start? — _open_
5. **Smoke-test overlap** — standalone S5, or fold the `--debug`/run-id/capsule knowledge into S1's verify-locally section only? — _open_
6. **Trigger style** — auto-fire on intent keywords (richer descriptions) vs explicit `/name` invocation? — _open_
7. **Home** — ✅ `jackin-dev` plugin (D4).
8. **Undocumented flows** — daily mental steps not written in the rule files that should become skills? — _open_

---

## 7. Decisions log

- **D1** — Build S1–S4 first (full specs in §5). S5–S8 parked until S1–S4 land.
- **D2** — S3 schema validator = `cargo xtask schema-check` (lives in the jackin' repo, reusable by CI, closes the PR/main-parity gap). Skill shells out to it.
- **D3** — S4 = thin orchestrator: adds only the 4 jackin'-specific gates, then delegates to `/code-review` + `pr-review-toolkit` agents.
- **D4** — Home = `jackin-dev` plugin, alongside `release*`, marketplace-installed.

Still open: Q4, Q5, Q6, Q8.

- **D5** — Build our **own** spec-driven workflow, OpenSpec-*inspired* (artifact roles + lifecycle discipline) and superpowers-*method* (brainstorming). Do **not** install OpenSpec: no `openspec/` dir, no JS CLI — it would duplicate the roadmap + docs and fight jackin's single-source / docs-as-spec rules.
- **D6** — **Archive = documentation.** When a feature ships, docs are updated and the roadmap item is retired into them (existing rule). Docs are the archive; no archive folder.
- **D7** — Release notes are **out of scope** for now. Possible future stage; not required.
- **D8** — Triggering is **explicit `/invoke` only** (no auto-fire). Slugs carry no `jackin-` prefix — the `jackin-dev:` plugin namespace is the prefix.
- **D9** — Any non-trivial scripting is **Rust**, compiled into a binary the skills shell out to — not shell or JS. Mechanical lifecycle ops (scaffold/strip roadmap item, validate sections, schema-check, trailers) become subcommands.
- **D10** — The opener is a **separate `propose` skill**, not `create-pr`. `propose` owns idea→branch→forks→roadmap-item scaffold, then chains into `create-pr`. `create-pr` is pure PR mechanics (reusable standalone for quick PRs).
- **D11** — The Rust lifecycle binary lives as **`cargo xtask` subcommands in the jackin' repo** (CI reuses them → PR/main parity; beside `pty-fixture`, `jackin-pr-trailers`, `pr prepare`).
- **D12** — `brainstorm` and `plan` are **two skills**: `brainstorm` fills `## Design`, `plan` breaks it into `## Tasks`.
- **D13** — `propose` is for **feature/idea work only** and **never writes code**. It collects everything into the roadmap item (branch + draft + early PR) and stops; implementation is always a separate later step. The brainstorm-vs-implement fork is removed from `propose`.
- **D14** — **Small fixes** (typo, dep bump, one-line fix, doc tweak) skip `propose` and use `create-pr` directly — no roadmap item.
- **D15** — Research reuses the existing built-in **`deep-research`** skill (no new research skill); `propose`/`brainstorm` call it on demand (`--research`).
- **D16** — Skills accept **arguments/flags** (e.g. `/jackin-dev:propose <idea> --auto-branch --no-pr`). Branch auto-naming is a per-call `--auto-branch` flag, not saved state.
- **D17** — **No jackin' commit skill.** `COMMITS.md` auto-loads; commits are made inline by `create-pr` / `goal` following it. The generic `commit-commands` plugin remains available for a standalone `/commit`.
- **D18** — PR body always uses the **full template, read from `.github/PULL_REQUEST_TEMPLATE.md` at runtime** — never embedded in a skill. Template change → skills follow automatically.
- **D19** — PR mechanics live in a shared Rust helper **`cargo xtask pr body`** (reads the template, auto-selects verify-locally blocks from the diff). `propose` and `create-pr` both call it and open their own PR — **no skill calls another skill.**
- **D20** — Implementation is the **`goal`** skill: `/goal Implement <slug>.md` builds a finalized roadmap item. Parked for later deep-design.
- **D21** — `brainstorm` runs as **freeform discussion** (not rigid Socratic), and writes decisions **incrementally into `## Design`** as each point settles (resumable across sessions).
- **D22** — `research` covers **both web and codebase**, and **folds findings into the existing Problem / Why / Design sections** (no separate `## Research` section to later strip). It wraps the built-in `deep-research` for the web half.
- **D23** — Boundary: `research` **gathers** (may add design *context/constraints*); `brainstorm` **decides** (writes the actual design choices). `brainstorm` may invoke `research` mid-design when it hits an unknown.
- **D24** — `research` is **optional and reorderable** in the feature pipeline; not every item needs it.

---

## 8. Workflow model — jackin'-native spec-driven pipeline

No new tooling. The **roadmap item `.mdx` is the living change-spec** while a change is open; on ship it retires into canonical docs.

Roadmap `.mdx` sections while a change is **active**:

| Section | Role (OpenSpec analogue) | State |
|---|---|---|
| `## Problem` | why / scope (proposal.md) | exists today |
| `## Why It Matters` | impact (proposal.md) | exists today |
| `## Design` | how / approach (design.md) | **added while active**, filled by brainstorm |
| `## Tasks` | checklist (tasks.md) | **added while active**, filled by plan |
| `## Related Files` | touch-points | exists today |
| `**Status**` | Open → Partially implemented → retired | exists today |

On ship: update user + contributor docs, strip `## Design`/`## Tasks`, retire the item per the existing retirement rule. **That is the archive.**

Lifecycle:

```
FEATURE / IDEA WORK
  propose            roadmap item draft + early PR  (NEVER codes; collects for the item, then stops)
    └► research      (optional) web + codebase; folds findings into Problem/Why/Design context
        └► brainstorm    freeform discussion; writes decisions incrementally into ## Design
            └► plan      fill ## Tasks
                └► goal   implement a FINALIZED roadmap item  (e.g. /goal Implement <slug>.md)
                    └► merge-pr   retire the item into docs (the archive)
  (research is optional + reorderable: brainstorm may call research mid-design when it hits an unknown)

SMALL FIX  (typo, dep bump, one-line fix, doc tweak)
  create-pr          branch + inline commit + PR, no roadmap item

PR mechanics shared by propose + create-pr: `cargo xtask pr body`
  reads .github/PULL_REQUEST_TEMPLATE.md + auto-selects verify-locally blocks from the diff.
  No skill calls another skill — both call the helper and open their own PR.
```

## 9. Skill pipeline (revised from §4)

**Pipeline skills** (the spec-driven front-to-back):

| Stage | Skill | Owns |
|---|---|---|
| open | `propose` | **feature/idea only, never codes.** idea → branch → scaffold roadmap item draft (`xtask change new`) → open early PR via `create-pr`. Collects everything for the item, then stops. |
| PR mechanics | `create-pr` | pure PR: variant classify, verify-block auto-select, 8-section body, heredoc `--body-file`, render self-check. The **small-fix path** (no roadmap item) and the PR-mechanics engine `propose` reuses. |
| research | `research` | (optional) reads the item, runs web (`deep-research`) + codebase exploration, folds findings into Problem/Why/Design context. Gathers, never decides. |
| design | `brainstorm` | freeform discussion; writes design decisions incrementally into `## Design`. May call `research` on an unknown. |
| plan | `plan` | break `## Design` → `## Tasks` |
| implement | `goal` | implement a **finalized** roadmap item: `/goal Implement <slug>.md`. Reads Problem/Why/Design/Tasks, builds it, updates docs. *(parked — deep-design later)* |
| finish | `merge-pr` | verify, reconcile, retire roadmap item into docs, squash-merge |

**Standalone skills** (unchanged, fire on their own intents): `review-pr`, `bump-schema`, `sync-docs`, `smoke-test`, `capsule-fixture`, `author-workflow`.

> §5's S1 spec is now split: `propose` owns pre-flight/branch/fork/roadmap-scaffold; `create-pr` owns classify/select/draft/create. Full per-skill specs land in §10 as we deep-dive each.

Borrowed: OpenSpec artifact roles + propose→archive discipline; superpowers brainstorming. Adapted onto jackin's existing roadmap + docs, no parallel structures.

---

## 10. Per-skill specs (deep-dive results)

Settled specs land here as we brainstorm each skill. Status 🟢 = agreed.

### 10.1 `propose` 🟢

**Purpose:** open a new **feature/idea** as a roadmap item plus an early PR. Never writes code — it collects everything needed for the roadmap item, then stops.

**Invocation:** `/jackin-dev:propose <idea or problem text> [flags]`

| Flag | Effect |
|---|---|
| `--branch <name>` | explicit branch name |
| `--auto-branch` | name the branch itself, no confirm |
| `--no-pr` | branch + roadmap draft only, no PR (rare; default always opens a PR) |
| `--research` | run `deep-research` first, fold findings into the draft |

**Flow:**
1. **Branch** — never commit to `main`. Derive `feature/<slug>` (or `fix/`,`refactor/`,`chore/` per change type) from the idea; suggest + confirm. Skip the confirm when `--auto-branch` or `--branch` given.
2. **(optional) Research** — on `--research`, call `deep-research`; summarize findings into the draft.
3. **Scaffold roadmap item** — `cargo xtask change new <slug>` creates `docs/content/docs/reference/roadmap/<slug>.mdx` with `**Status**: Open`, `## Problem`, `## Why It Matters`, empty `## Design`, `## Tasks`, `## Related Files`; adds the sidebar entry under Roadmap → Open items in `astro.config.ts`; runs the sidebar/overview audit. Fill Problem / Why It Matters from the idea text.
4. **Commit + push** — `docs:` type, DCO `-s`.
5. **Open PR** — immediately, unless `--no-pr`, via `create-pr` mechanics. Summary = the idea; What ships = "roadmap item draft for `<slug>`"; Verify = docs render.
6. **Stop.** Point the user at next steps: `/jackin-dev:brainstorm <slug>` to fill `## Design`.

**Never:** write source code, run an implementation, or fill `## Design` / `## Tasks` (those are `brainstorm` / `plan`).

**xtask needed:** `change new`.

**Open thread:** if you already have a roadmap-item skill whose behavior should carry over, point me at it; otherwise `propose` supersedes it.

### 10.2 `create-pr` 🟢

**Purpose:** open a PR for a **small fix** (no roadmap item), and serve as the shared PR-mechanics path. Commits inline; no separate commit skill.

**Invocation:** `/jackin-dev:create-pr [flags]`

| Flag | Effect |
|---|---|
| `--branch <name>` / `--auto-branch` | branch name explicit / self-chosen (same as `propose`) |
| `--title <msg>` | commit + PR title override (else derived from the diff, Conventional Commits) |

**Flow (state-checking, no "modes"):**
1. **Branch** — if on `main`, create a `fix/`,`chore/`,`docs/`,`refactor/`-prefixed branch (suggest + confirm unless `--auto-branch`/`--branch`). Never commit to `main`.
2. **Commit** — if there are uncommitted changes, commit them **inline** per `COMMITS.md` (Conventional Commits, DCO `-s`); if already committed, skip. Push.
3. **Body** — `cargo xtask pr body` reads `.github/PULL_REQUEST_TEMPLATE.md` and auto-selects verify-locally blocks from the changed paths (Checkout + isolation always; Rust/Docs/User-smoke/capsule/schema conditional). Model fills the prose sections (Summary, What ships).
4. **Create** — `gh pr create --body-file`; then render self-check (`gh pr view --json body`); fix `\`` / `\$` artifacts via `gh pr edit` if present.
5. **Reply** — share URL + repeat the verify-locally commands.

**xtask needed:** `pr body` (shared with `propose`).

**Not for:** feature/idea work (use `propose`); implementation (use `goal`).

### 10.3 `research` 🟢

**Purpose:** enrich an **existing** roadmap item with gathered evidence — external prior art and the current jackin' codebase reality — folded into its Problem / Why / Design context. Gathers, never decides.

**Invocation:** `/jackin-dev:research <slug> [flags]`

| Flag | Effect |
|---|---|
| `--web-only` | skip codebase exploration |
| `--codebase-only` | skip the web pass |
| `--question "<q>"` | focus the research on a specific question instead of the whole item |

**Flow:**
1. **Read the item** — load `docs/content/docs/reference/roadmap/<slug>.mdx`; extract Problem / Why / open questions.
2. **Web pass** — call the built-in **`deep-research`** skill with the item's questions; collect cited findings (prior art, libraries per `ENGINEERING.md` crate-choice criteria, approaches other projects took).
3. **Codebase pass** — explore how jackin' does the relevant thing today (Explore/grep; map related files, current behavior, constraints). Feeds `## Related Files`.
4. **Fold in** — distribute findings into **Problem** (sharper statement), **Why It Matters** (evidence), and **Design** as *context/constraints only* (e.g. "crate X is the maintained choice", "current code path is `src/...`"). Do **not** write design decisions — that's `brainstorm`.
5. **Cite** — keep source links inline so `brainstorm`/review can audit. (These are contributor-facing roadmap details; the item retires into docs on ship.)
6. **Commit + push** — `docs:`; report what was added.

**Calls:** `deep-research` (web). **Does not** decide design or write `## Tasks`.

### 10.4 `brainstorm` 🟢

**Purpose:** turn an item's intent + gathered research into concrete **design decisions** in `## Design`, through freeform discussion.

**Invocation:** `/jackin-dev:brainstorm <slug> [flags]`

| Flag | Effect |
|---|---|
| `--research` | run `research` first if the item looks thin |
| `--resume` | continue from the current `## Design` state |

**Flow:**
1. **Load context** — read the item (Problem/Why/Design-so-far/Related Files) + any research already folded in.
2. **Discuss freeform** — open back-and-forth: surface alternatives, trade-offs, open questions. No rigid script. Pull in jackin' design principles + relevant rule files (`ENGINEERING.md`, `HOST_AND_CONTAINER.md`, TUI/docs rules) as constraints.
3. **Write incrementally** — as each point settles, append/update the matching part of **`## Design`** in the `.mdx` (decision + the one-line *why*). Resumable: a later `--resume` picks up where Design left off.
4. **Hit an unknown?** — invoke `research --question "<q>"` to gather, then resume deciding.
5. **Converge** — when Design covers the approach end-to-end, summarize and point at `/jackin-dev:plan <slug>` to break it into `## Tasks`.
6. **Commit + push** — `docs:` per settled chunk (push-after-commit).

**Boundary:** writes **decisions** (Design), not gathered evidence (that's `research`) and not the task breakdown (that's `plan`). Never writes source code.

**xtask needed:** none beyond `change` helpers; both skills edit the `.mdx` directly and commit.
