# jackin-dev workflow

How the spec-driven workflow skills are structured and why. Stable reference — the per-skill behavior lives in each `skills/<name>/SKILL.md`; this file records the architecture and the decisions behind it.

## Design principle

**Skills are the orchestration layer; the jackin' repo's topic files stay the source of truth.** Each skill is a triggered checklist that sequences steps scattered across the repo's auto-loaded rule files (`PULL_REQUESTS.md`, `BRANCHING.md`, `COMMITS.md`, `PRERELEASE.md`, …) and shells out to validator scripts where a step is mechanical. It names the governing rule file instead of copying it, so when a rule changes it changes in one place and the skill keeps pointing at it. This mirrors jackin's own single-source / symlink-never-copy ethos and keeps the skills thin and rot-resistant.

The repo's rule files already auto-load into context. What they do not give is **procedural sequencing on demand** — an ordered, fail-closed checklist that walks a multi-step flow. That gap is what these skills fill.

## Workflow model

The roadmap item `.mdx` is the living change-spec while a change is open; on ship it retires into canonical docs (the archive). Roadmap sections while active: `## Problem` / `## Why It Matters` (exist today), `## Design` (filled by `brainstorm`), `## Tasks` (filled by `plan`, deferred), `## Related Files`, `**Status**`.

```
FEATURE / IDEA
  propose      roadmap item draft + early PR (never codes; collects, then stops)
  brainstorm   freeform discussion → writes decisions into ## Design
  /goal Implement <slug>.md   build the finalized item (external spec-runner)
  merge-pr     pre-merge gate → retire item into docs → squash-merge

RESEARCH   research → author prompt.mdx brief → /goal Follow <brief> → dossier under docs/content/docs/research/<slug>/

SMALL FIX  create-pr   branch + inline commit + PR, no roadmap item
```

`goal` is the universal spec-runner (external, not built here): a roadmap item → code; a research brief → dossier. PR mechanics are shared by `propose` + `create-pr`; no skill calls another skill.

## The five skills

| Skill | Role |
|---|---|
| `propose` | open a feature/idea → roadmap item draft + early PR. Never codes. |
| `brainstorm` | freeform design discussion → writes `## Design`. |
| `research` | standalone multi-page research dossier on the docs site (web + codebase). |
| `create-pr` | small-fix PR + the shared PR-mechanics path (template + verify-block auto-select). |
| `merge-pr` | pre-merge gate (blast-radius confirm, CI poll, roadmap retirement) + squash-merge. |

Released separately in this plugin: `release`, `release-check`, `release-notes`.

## Manual-only

Every skill sets `disable-model-invocation: true` — the platform lever that stops auto-firing — and each `description` is phrased so the trigger is the explicit `/jackin-dev:<name>` invocation, never an intent keyword. No jackin-dev skill auto-fires. On agents lacking the frontmatter field, the explicit-invocation description is the fallback guarantee.

## Cross-agent packaging

jackin' launches **claude, codex, amp, kimi, opencode**. `SKILL.md` is a portable standard, so one `skills/<name>/SKILL.md` source serves all of them; only the install path differs:

- **claude** — `.claude-plugin/plugin.json` + `jackin-marketplace` (`/plugin install`).
- **codex** — `.codex-plugin/plugin.json` (`"skills": "./skills/"`) + the `/plugins` flow.
- **codex / amp / opencode / kimi** all read `~/.agents/skills/`. Installing the tree there once (e.g. `skills add jackin-project/jackin-dev -s '*' -a <agent> --global`) covers all four. This is how the `jackin-the-architect` role image vendors the skills for the non-Claude agents.

Author to the portable spec: required `name` / `description` frontmatter, markdown body (~100 lines, 500 hard cap), reference files one level deep, forward-slash paths, Rust-binary scripts shelled out by path. Match specificity to fragility — fixed sequences (squash `(#N)` + trailers, capsule ordering, schema gate) get exact commands; judgment tasks (`brainstorm`, PR prose) describe outcomes and let the agent adapt.

## xtask helpers

Mechanical lifecycle ops live as `cargo xtask` subcommands in the jackin' repo (CI reuses them → PR/main parity). Built: `change new`, `research scaffold`, `research check`, `roadmap audit`. Referenced by the skills but not yet built — the skills perform these steps directly until they exist: `pr body` (template read + verify-block selection), `roadmap retire` (mechanical parts of retirement), `schema-check` (5-artifact migration gate).

## Deferred

- `plan` skill — break `## Design` into `## Tasks`. Not in the build set yet.
- `pr body`, `roadmap retire`, `schema-check` xtasks — see above.
- Cross-agent native manifests beyond claude/codex (opencode/kimi/amp install via `~/.agents/skills/` for now).
