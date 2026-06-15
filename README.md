# jackin-dev

Development workflow plugin for the [jackin](https://github.com/jackin-project/jackin) project. Skills for the spec-driven feature workflow, pull requests, research, and release management.

**Skills are the orchestration layer; the jackin' repo's rule files stay the source of truth.** Each skill sequences steps that are otherwise scattered across the repo's auto-loaded topic files (`PULL_REQUESTS.md`, `BRANCHING.md`, `COMMITS.md`, `PRERELEASE.md`, …) and names the governing rule instead of copying it — so when a rule changes, the skill keeps pointing at it. The full behavior of each skill lives in its `skills/<name>/SKILL.md`; this README is the map.

All skills are **manual-only** — invoke each explicitly as `/jackin-dev:<name>`; none auto-fire (each sets `disable-model-invocation: true`).

## Workflow

The roadmap item `.mdx` is the living change-spec while a change is open; on ship it retires into canonical docs (the archive).

```
FEATURE / IDEA
  propose      roadmap item draft + early PR  (never codes; collects, then stops)
  brainstorm   freeform discussion → writes decisions into ## Design
  /goal Implement <slug>.md   build the finalized item  (external spec-runner)
  merge-pr     pre-merge gate → retire item into docs → squash-merge

RESEARCH
  research → author prompt.mdx brief → /goal Follow <brief> → dossier under docs/content/docs/research/<slug>/

SMALL FIX
  create-pr → merge-pr   (branch + inline commit + PR, no roadmap item)
```

`goal` is the universal spec-runner (external, used not built here): a roadmap item → code; a research brief → dossier. PR mechanics are shared by `propose` + `create-pr`; no skill calls another skill.

## Feature skills

### `propose`
Open a feature/idea as a tracked **roadmap item + early PR**. Scaffolds `reference/roadmap/<slug>.mdx` (`Problem` / `Why It Matters` / `Design` / `Tasks` / `Related Files`) + its sidebar entry, fills Problem/Why from the idea, opens the PR, and **stops** — it never writes code or fills `Design`/`Tasks`.
*Flags:* `--branch <name>` / `--auto-branch`, `--no-pr`, `--research`. *Not for:* small fixes (→ `create-pr`).

### `brainstorm`
Turn an item's intent into concrete **design decisions** through freeform back-and-forth, written incrementally into `## Design` (decision + one-line why; resumable). Pulls in jackin' design principles and rule files as constraints. Never codes; never fills `Tasks`.
*Flags:* `--research`, `--resume`.

### `research`
Produce a standalone **multi-page dossier** under `docs/content/docs/research/<slug>/` — `index.mdx` + `prompt.mdx` brief + numbered chapters. Gathers web (via `deep-research`) + codebase evidence; every external claim sourced, every local number method-stamped. Brief-driven: author the brief, then execute via `/goal Follow <brief>`. Gathers — does not make design decisions.
*Flags:* `--brief-only`, `--in-roadmap` (small research only), `--web-only` / `--codebase-only`.

### `create-pr`
Open a PR for a **small fix** (typo, dep bump, one-line bugfix, doc tweak) — no roadmap item. Also the shared PR-mechanics path: branch, inline commit (Conventional Commits, DCO `-s`), body from `.github/PULL_REQUEST_TEMPLATE.md` with verify-locally blocks auto-selected from the changed paths.
*Flags:* `--branch <name>` / `--auto-branch`, `--title <msg>`. *Not for:* feature work (→ `propose`).

### `merge-pr`
Run the jackin' pre-merge gate **fail-closed**, retire the roadmap item into docs, then squash-merge. Gates: blast-radius confirm (schema / auth / workflow / force-push), poll CI until green, roadmap retirement (retire-into-docs or status move), metadata reconcile, squash with `(#N)` + trailers.
*Flags:* `--no-poll`, `--admin <check>`.

## Release skills

| Skill | Description |
|---|---|
| `release-check` | Pre-release validation: CI status, tests, clippy, fmt, doc links, TODOs, security exceptions, Docker build |
| `release-notes` | Generate the `[Unreleased]` changelog from merged PRs since the last tag (Keep a Changelog format) |
| `release` | Full orchestrator: `release-check` → notes → version recommendation → confirm → `cargo release` |

## Installation

`SKILL.md` is a portable standard — one `skills/<name>/` source serves every agent jackin' supports; only the install path differs.

### Claude Code

```
/plugin marketplace add jackin-project/jackin-marketplace
/plugin install jackin-dev@jackin-marketplace
```

### Codex, Amp, OpenCode, Kimi

All four read `~/.agents/skills/`. Install the tree there once with the [`skills`](https://www.npmjs.com/package/skills) CLI (per agent, since agent CLIs aren't auto-detected in every environment):

```sh
npx -y skills add "jackin-project/jackin-dev" -s '*' -a codex --global --yes
npx -y skills add "jackin-project/jackin-dev" -s '*' -a amp   --global --yes
```

`--global` writes the canonical `~/.agents/skills/<skill>/` tree, which OpenCode and Kimi read from the same path. Codex additionally reads `.codex-plugin/plugin.json` (the `/plugins` flow); Amp also offers `amp skill add jackin-project/jackin-dev`. Pin to a release tag in production (`jackin-project/jackin-dev#vX.Y.Z`).

The [`jackin-the-architect`](https://github.com/jackin-project/jackin-the-architect) role image bakes this in, so every agent it launches has the skills with no per-agent step.

## How it works

- **Manual-only** is enforced with `disable-model-invocation: true` frontmatter, with the explicit-invocation `description` as the fallback for agents lacking that field.
- **Mechanical steps shell out to `cargo xtask` subcommands** in the jackin' repo (CI reuses them → PR/main parity). Built: `change new`, `research scaffold`, `research check`, `roadmap audit`. Referenced but not yet built — the skills do these steps directly until they exist: `pr body`, `roadmap retire`, `schema-check`.
- **Deferred:** the `plan` skill (break `## Design` → `## Tasks`).

## Requirements

Designed for the `jackin-project/jackin` repository. Expects `gh` authenticated; the release skills additionally expect `cargo-release` installed, a `release.toml` with `pre-release-replacements` for `CHANGELOG.md`, and a `.github/workflows/ci.yml`.

## License

Apache-2.0
