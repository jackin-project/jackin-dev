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
- **Description:** third person, states **what**, ≤1024 chars. **For jackin-dev: the trigger is the explicit `/invoke` itself (D8) — do NOT pack intent keywords that auto-fire the skill.** Phrase as "Use when the operator runs `/jackin-dev:<name>`."
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

## 4. Final build set

The build set is the **5 spec-driven pipeline + research skills**. The earlier broader candidate list (smoke-test, sync-docs, capsule-fixture, author-workflow, review) is **dropped** — those flows stay covered by the auto-loaded rule files and existing tooling (`/code-review`, `/verify`, `cargo xtask pty-fixture`, etc.).

| Skill | Role | Spec |
|---|---|---|
| `propose` | open a feature/idea → roadmap item draft + early PR (never codes) | §10.1 🟢 |
| `research` | standalone research dossier on the docs site (web + codebase) | §10.3 🟢 |
| `brainstorm` | freeform design discussion → writes `## Design` | §10.4 🟢 |
| `create-pr` | small-fix PR + shared PR-mechanics (template + verify-block auto-select) | §10.2 🟢 |
| `merge-pr` | pre-merge gate + roadmap retirement + squash-merge | §10.5 🟢 |

External (referenced, not built here): `goal` (universal spec-runner), `deep-research` (built-in, called by `research`). Commits are inline per `COMMITS.md` — no commit skill.

Full per-skill specs live in §10; the pipeline in §8–§9. Sections 5 (old gerund specs) removed as superseded.

---

## 6. Open questions

All major scope questions resolved (see Decisions log). Remaining:

- **Q-plan** — `plan` DEFERRED out of the build set: break `## Design` → `## Tasks`. Revisit/improve later.
- **Q-research-final** — `research` output format/placement matches jackin #583 dossier; confirm once the operator finalizes a second real example.
- **Q-build-order** — which of the 5 to author first. Suggested: `create-pr` (smallest, shared mechanics) → `propose` → `merge-pr` → `brainstorm` → `research`.

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
- **D8** — Triggering is **explicit `/invoke` only — HARD RULE, no exceptions.** **No jackin-dev skill may ever auto-fire.** The operator invokes every skill manually and only one way: `/jackin-dev:<name>`. Authoring consequence: each SKILL.md `description` must be written so the **trigger is the explicit invocation itself** (e.g. "Use when the operator runs `/jackin-dev:propose`. …") — describe what the skill does, but do **not** pack intent keywords that would make the harness auto-select it. Slugs carry no `jackin-` prefix — the `jackin-dev:` plugin namespace is the prefix.
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
- **D22** — ~~`research` folds findings into Problem / Why / Design.~~ **REVERSED → FINAL (see D25).** `research` produces a **standalone multi-page dossier** published on the docs website. Reference: jackin `docs/content/docs/research/token-optimization-research/` (PR #583): `meta.json` + `index.mdx` + `prompt.mdx` (the brief) + numbered chapters + optional `tools/`. Covers web (`deep-research`) + codebase.
- **D25** — Research **storage**: default = **separate dossier folder** `docs/content/docs/research/<slug>/`; **inline in the roadmap item only on explicit request** (`--in-roadmap`) and only for small research. Big research always gets its own folder.
- **D26** — **`goal` is the universal spec-runner.** `/goal Follow <path>` (or `/goal Implement <path>`) executes any self-contained spec file end-to-end: a **roadmap item** → implement code; a **research brief** (`prompt.mdx`) → produce the dossier. Same skill, the spec decides the work. (The token-optimization brief documents its own run line as `/goal Follow token-optimization-research.md`.)
- **D27** — Research sidebar wiring (`meta.json` per dossier + parent `research/meta.json`) is mechanical → **xtask candidates** `research scaffold <slug>` and `research check` (pages[] match files on disk).
- **D28** — `merge-pr` **authorization**: invoking the skill is the per-PR go for normal PRs (proceed through the gates). **Pause for one explicit confirm only on high-blast-radius PRs** — schema-version bump, security/auth surface, CI/workflow changes, or anything needing a force-push / `--admin`.
- **D29** — `merge-pr` **does the roadmap retirement** as a pre-merge action: if the PR ships the **last** piece of the item, run the 7-step retire-into-docs (move remaining content to canonical docs, delete the `.mdx`, update `roadmap/index.mdx`, sidebar/overview audits); if only **partial**, move `**Status**` to *Partially implemented* with remaining phases named. Feature/user docs themselves are written during implementation (`goal`); `merge-pr` owns the roadmap-item retirement specifically.
- **D30** — `merge-pr` **polls CI until green**, then merges; stops if any check fails (no `--admin` bypass without explicit per-failure opt-in).
- **D31** — `merge-pr` **trusts CI green** (`gh pr checks` all pass) as the gate; no local fmt/clippy/nextest re-run.
- **D32** — **Drop `review-pr` and `goal` from the build set.** `review-pr` is not built — use `/code-review` (and `/code-review ultra`) directly. `goal` is an **existing external skill** (the universal spec-runner `/goal Implement <item> | Follow <brief>`), referenced by the pipeline + `research` but **not authored in `jackin-dev`**.
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
    └► brainstorm    freeform discussion; writes decisions incrementally into ## Design
        └► goal Implement <slug>.md   build the finalized roadmap item (external runner)
            └► merge-pr   retire the item into docs (the archive)
  (plan — break ## Design into ## Tasks — DEFERRED; add later)

RESEARCH  (standalone dossier; default home = docs/content/docs/research/<slug>/)
  research          author brief (prompt.mdx) ──► goal Follow <brief>  ──► dossier published to docs
  (small research may be stored --in-roadmap instead; big research always a separate folder)

SMALL FIX  (typo, dep bump, one-line fix, doc tweak)
  create-pr          branch + inline commit + PR, no roadmap item

goal = universal spec-runner: /goal {Implement|Follow} <spec>. Roadmap item → code; research brief → dossier.

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
| design | `brainstorm` | freeform discussion; writes design decisions incrementally into `## Design`. |
| run | `goal` *(external — not built here)* | the universal spec-runner you already use: `/goal Implement <slug>.md` builds a finalized roadmap item; `/goal Follow <brief>` executes a research brief. Referenced by the pipeline, authored elsewhere. |

Adjacent (not in the linear feature path):

| Skill | Owns |
|---|---|
| `research` | author a research **brief** (`prompt.mdx`) and produce a standalone **dossier** under `docs/content/docs/research/<slug>/` (web `deep-research` + codebase). Executed via `goal`. Default separate folder; `--in-roadmap` for small. |
| finish | `merge-pr` | verify, reconcile, retire roadmap item into docs, squash-merge |

**Standalone skills** (unchanged, fire on their own intents): `review-pr`, `bump-schema`, `sync-docs`, `smoke-test`, `capsule-fixture`, `author-workflow`.

> §5's S1 spec is now split: `propose` owns pre-flight/branch/fork/roadmap-scaffold; `create-pr` owns classify/select/draft/create. Full per-skill specs land in §10 as we deep-dive each.

Borrowed: OpenSpec artifact roles + propose→archive discipline; superpowers brainstorming. Adapted onto jackin's existing roadmap + docs, no parallel structures.

---

## 10. Per-skill specs (deep-dive results)

Settled specs land here as we brainstorm each skill. Status 🟢 = agreed.

### 10.1 `propose` 🟢 — BUILT (`skills/<name>/SKILL.md`)

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

### 10.2 `create-pr` 🟢 — BUILT (`skills/<name>/SKILL.md`)

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

### 10.3 `research` 🟢 — BUILT (`skills/<name>/SKILL.md`)

**Purpose:** produce a **standalone research dossier** — a multi-page deliverable published on the docs site — for an open question or roadmap topic. Brief-driven and executed end-to-end. Gathers + synthesizes evidence (web + codebase); does not make product design decisions.

**Reference implementation:** jackin `docs/content/docs/research/token-optimization-research/` (PR #583).

**Canonical output layout** (fumadocs):

```
docs/content/docs/research/<slug>/
├── meta.json          # { title, defaultOpen:false, pages:[...] }  — sidebar order
├── index.mdx          # dossier landing: headline numbers, how-to-read, tier list
├── prompt.mdx         # the research BRIEF that was run (the spec; carries the /goal run line)
├── NN-*.mdx           # numbered chapters (00 summary, 01.. foundations, 10-20 areas, 30+ synthesis)
└── tools/             # optional: scripts (count_tokens.py ...) + own meta.json + index.mdx
```
Parent `docs/content/docs/research/meta.json` lists each dossier under `pages`.

**Two-part shape:**
1. **Author the brief** (`prompt.mdx`) — the full self-contained spec: mission, constraints, operating rules, the exact chapter list (§10 of the brief), and the evidence bar (every external claim sourced, every local number method-stamped). This is the durable, reviewable artifact.
2. **Execute the brief** — run it (typically via `goal`, see D26) to produce `index.mdx` + chapters + `tools/`, autonomously, without per-step questions.

**Invocation:** `/jackin-dev:research <slug> [flags]`

| Flag | Effect |
|---|---|
| `--brief-only` | author `prompt.mdx` (the spec) and stop; execute later via `goal` |
| `--in-roadmap` | store findings **inline in the roadmap item** instead of a separate dossier (small research only) |
| `--web-only` / `--codebase-only` | restrict gathering to one pass |

**Storage decision (D25):**
- **Default → separate dossier folder** under `docs/content/docs/research/<slug>/`.
- **Inline in the roadmap item** only when the operator explicitly asks (`--in-roadmap` or "put it in the roadmap item"), and only for **small** research. Big research always gets its own folder.

**Flow:**
1. **Scope** — from the topic/roadmap item, draft (or load) the brief `prompt.mdx`: mission, chapter list, evidence rules.
2. **Gather** — web via built-in `deep-research`; codebase via Explore/grep. Every external claim carries a source URL; every local number carries its method.
3. **Write** — `index.mdx` (headline numbers + how-to-read + tier list) and the numbered chapters in a fixed per-technique record schema; bundle any reproduction scripts under `tools/`.
4. **Wire sidebar** — create/update the dossier `meta.json` and the parent `research/meta.json` (candidate `xtask research scaffold` / `research check`, D27).
5. **Docs gate + commit** — run the `bun` docs gate; commit `docs(research):`; push.

**Calls:** `deep-research` (web) and, for execution, `goal`. **Does not** write code or design decisions.
**xtask candidates:** `research scaffold <slug>` (create folder + meta.json + brief stub), `research check` (validate meta.json pages match files on disk).

### 10.4 `brainstorm` 🟢 — BUILT (`skills/<name>/SKILL.md`)

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
4. **Hit an unknown?** — gather quickly (Explore/grep, or `deep-research`); for a large investigation, spin a separate `research` dossier and link it from `## Design`. Then resume deciding.
5. **Converge** — when Design covers the approach end-to-end, summarize and point at `/jackin-dev:plan <slug>` to break it into `## Tasks`.
6. **Commit + push** — `docs:` per settled chunk (push-after-commit).

**Boundary:** writes **decisions** (Design), not gathered evidence (that's `research`) and not the task breakdown (that's `plan`). Never writes source code.

**xtask needed:** none beyond `change` helpers; both skills edit the `.mdx` directly and commit.

### 10.5 `merge-pr` 🟢 — BUILT (`skills/<name>/SKILL.md`)

**Purpose:** run the jackin' pre-merge gate and squash-merge a PR, including roadmap-item retirement. Fail-closed; reuses `.github/` agent rules (it sequences them, doesn't restate them).

**Invocation:** `/jackin-dev:merge-pr [<PR>] [flags]` (defaults to the PR for the current branch)

| Flag | Effect |
|---|---|
| `--no-poll` | don't wait on pending CI; stop and report instead |
| `--admin <check>` | authorize bypassing one named failing check (still requires the high-blast-radius confirm) |

**Flow (fail-closed; STOP = halt + report):**
1. **Resolve PR** — current branch's PR (or the arg). Read `gh pr view` + `gh pr diff`.
2. **Blast-radius classify (D28)** — high if the diff touches a versioned schema, auth/security surface, `.github/workflows/**`, or needs force-push/`--admin`. High → **pause for one explicit confirm**; normal → proceed (invoking the skill is the go).
3. **CI gate (D30/D31)** — `gh pr checks`: if pending, **poll until green** (unless `--no-poll`); if any **fail**, STOP (no `--admin` bypass unless `--admin <check>` + confirm). Trust CI; no local re-run.
4. **Roadmap retirement (D29)** — does this PR ship the **last** piece of its roadmap item?
   - **Yes** → run the 7-step retire-into-docs: move remaining operator detail to `guides/`/`commands/`, design detail to `reference/`, delete the roadmap `.mdx`, replace with a Completed bullet in `roadmap/index.mdx`, repoint inbound links, run sidebar + overview audits, run the `bun` docs gate. Commit `docs:`; push.
   - **Partial** → set `**Status**: Partially implemented` with remaining phases named; sidebar/overview audits. Commit; push.
   - candidate `xtask roadmap retire <slug>` / `roadmap audit` for the mechanical parts.
5. **Metadata reconcile** — title/body match the final diff? Squash writes the title verbatim into history. Fix via `gh pr edit` if stale; surface the change in the reply.
6. **Squash-merge** — build body file (prose summary, no checklists), append trailers via `cargo run -p jackin-pr-trailers -- --body-file`, then `gh pr merge <PR> --squash --body-file`. Title must carry `(#N)`.
7. **Report** — merged SHA + what retirement did.

**Bundles:** `jackin-pr-trailers`. **xtask candidates:** `roadmap retire`, `roadmap audit`.

---

## 11. Cross-agent packaging (all agents jackin' supports)

jackin' mounts credentials for **claude, codex, amp, kimi, opencode** — skills must work on all five, not Claude only. Good news: **SKILL.md is a portable open standard** (agentskills.io) — every one of these agents reads the same `SKILL.md` format. Only the install path and plugin system differ.

### Per-agent targets

| Agent | Project skills dir | Native plugin system | jackin-dev delivery |
|---|---|---|---|
| **claude** (Claude Code) | `.claude/skills/` | **Yes** — `.claude-plugin/plugin.json` + marketplace | native plugin (already present) |
| **opencode** | `.opencode/skill/` *(singular)* | **Yes** — `.opencode/plugin/` + `opencode.json` config | native plugin manifest + skills |
| **amp** | `.agents/skills/` (also reads `.claude/skills/` for compat) | No | skills dir + `AGENTS.md` reference |
| **codex** | `.codex/skills/` | No | skills dir (symlink) + `AGENTS.md` reference |
| **kimi** (Kimi Code CLI) | project skills dir (Project>User>Extra>Built-in) | No | skills dir + MCP |

### Decisions

- **D33** — **One source of truth, many targets.** Author each skill once as `skills/<name>/SKILL.md` in portable SKILL.md format (required frontmatter `name` + `description`, markdown body, sidecar reference files). Keep any agent-specific behavior out of the body; distribute the same file to each agent's path.
- **D34** — **Provide a native plugin where the agent has one.** ✅ SHIPPED: `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json`, `.opencode/INSTALL.md`, `.codex/INSTALL.md`, README install docs for all 5 agents (amp/kimi via AGENTS.md + skills dir). Claude: `.claude-plugin/plugin.json` (have it). OpenCode: add an OpenCode plugin/config manifest. Codex / Amp / Kimi: no plugin system → deliver via the skills directory + an `AGENTS.md` pointer (jackin-dev already ships `.codex/INSTALL.md`; extend the same pattern to amp/kimi).
- **D35** — **Distribution = per-agent native plugin manifests + marketplaces, following the `obra/superpowers` model.** No custom sync binary. One `skills/<name>/SKILL.md` source of truth; ship an agent-specific manifest at repo root for each agent that has a plugin system, and let each agent install jackin-dev through its own channel:
  - **claude** → `.claude-plugin/plugin.json` (have) → **`jackin-project/jackin-marketplace`** (already owned) → `/plugin install jackin-dev@jackin-marketplace`.
  - **codex** → `.codex-plugin` manifest → Codex `/plugins` install (Codex now has a plugin/marketplace flow).
  - **opencode** → `.opencode/` manifest + install instructions.
  - **kimi** → skills marketplace / skills dir.
  - **amp** → no plugin; AGENTS.md compat (reads `.claude/skills/`) + the `.codex/INSTALL.md`-style pointer.
  Superpowers ships exactly this set (`.claude-plugin`, `.codex-plugin`, `.cursor-plugin`, `.opencode/`, `gemini-extension.json`) over one `skills/` tree; jackin-dev mirrors it for jackin's five agents (drop cursor/gemini). Supersedes the earlier custom-`xtask skills sync` idea.
- **D36** — **Invocation is not identical across agents.** The manual `/jackin-dev:<name>` namespaced slash command is a Claude-plugin convention. On codex/amp/kimi/opencode the trigger may be the bare skill name or "read and follow `skills/<name>/SKILL.md`". The **manual-only rule (D8) still holds everywhere**; only the literal trigger string differs. Each SKILL.md description stays explicit-invocation-oriented so no agent auto-fires it.

- **D37** — **Build set = 5 skills** (`propose`, `research`, `brainstorm`, `create-pr`, `merge-pr`). Dropped: the 5 standalones (`smoke-test`, `sync-docs`, `capsule-fixture`, `author-workflow`), `review-pr`, and **`plan`** (deferred — break `## Design` → `## Tasks`, add later). Dropped flows stay covered by auto-loaded rules + existing tooling.

### Authoring consequence
Write every skill to the **portable SKILL.md spec** (not Claude-only features): required `name`/`description` frontmatter, markdown body, reference files one level deep, forward-slash paths, Rust-binary scripts shelled out by path. Avoid Claude-plugin-only constructs in the skill body so the same file works unmodified on all five agents.

---

## 12. Authoring standard for the 5 skills (efficiency + predictability)

Synthesized from Anthropic's skill best-practices, `obra/superpowers` `writing-skills`, and current field guides. These rules make our skills lean, predictable, and portable. Each becomes a checklist item when we author a SKILL.md.

### Manual-only, the right way (upgrades D8)
- **D38** — Enforce manual-only with **frontmatter**, not just wording: set **`disable-model-invocation: true`** on every jackin-dev SKILL.md (the platform lever that stops auto-firing). Keep the description explicit as the cross-agent fallback for agents lacking that field. This is the proper mechanism behind D8 — descriptions alone are a weaker guarantee.
- Description still phrased third-person "Use when the operator runs `/jackin-dev:<name>` …", **what + when only — never summarize the workflow** (workflow lives in the body, loaded on trigger).

### Standard SKILL.md skeleton (all 5)
- **D39** — Every skill follows one skeleton for predictability:
  1. **frontmatter** — `name`, `description` (≤1024 chars), `disable-model-invocation: true`.
  2. **Overview** — 1–2 sentence core purpose.
  3. **When to use / When NOT to use** — include the negative cases (e.g. `propose` not for small fixes; `create-pr` not for features). Testing both directions is a best-practice.
  4. **Flags** — table, executable args early.
  5. **Process** — the ordered flow.
  6. **Common mistakes** — the rationalizations to block.
  Body **target ~100 lines, ≤500 hard cap** (mattpocock: aim ~100, split past 500); reference files one level deep; forward-slash paths.
- **Description = two sentences** (mattpocock): sentence 1 states *what*; sentence 2 = "Use when the operator runs `/jackin-dev:<name>`." No keyword blob.

### Degrees of freedom — resolve the "don't over-specify" tension
- **D40** — Match specificity to fragility (the one rule that reconciles "describe outcomes, not steps" with our fixed sequences):
  - **Fragile / order-dependent / consistency-critical** → exact steps + push to a **Rust binary** (capsule `--capsule` ordering, squash `(#N)` + trailers, the 5-artifact schema gate, roadmap retirement). "Do not modify the command."
  - **Judgment** → describe the outcome, give constraints, let the agent adapt (`brainstorm` discussion, PR prose sections). Don't script these.

### Predictability patterns to apply
- **Fail-closed gates as explicit STOPs** — `merge-pr` lists each gate as a hard STOP with the rule it enforces; add a short "Red flags — STOP" list (superpowers pattern) for the auth/CI/retirement gates.
- **Flowcharts only for real decision points / loops** — `create-pr` change-classification, `merge-pr` gate sequence. Never for linear steps or reference.
- **Checklists for multi-step flows** — copy-into-response checklist for `propose` and `merge-pr` so progress is trackable.
- **Cross-reference, don't inline** — point at rule files / other skills by name (`REQUIRED BACKGROUND: …`); never `@`-force-load (wastes context). Consistent with §3.
- **Conciseness** — only add what the agent doesn't already know; the auto-loaded rule files carry the rest. Trim every sentence that exists in two places.

### Testing before ship
- **D41** — Author each skill against failing scenarios first (RED→GREEN): run the target flow **without** the skill, capture the wrong behavior, write the minimal skill that fixes exactly that, re-test with the skill present including **non-activation** cases. No skill ships untested.
