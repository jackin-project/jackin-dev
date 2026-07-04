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
Open a feature/idea as a tracked **roadmap item + early PR**. Scaffolds `roadmap/<slug>.mdx` (`Problem` / `Why It Matters` / `Design` / `Tasks` / `Related Files`) + its sidebar entry, fills Problem/Why from the idea, opens the PR, and **stops** — it never writes code or fills `Design`/`Tasks`.
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

The agents below install via the [`skills`](https://www.npmjs.com/package/skills) CLI (per agent, since agent CLIs aren't auto-detected in every environment). Pin to a release tag in production (`jackin-project/jackin-dev#vX.Y.Z`).

### Codex

```sh
npx -y skills add "jackin-project/jackin-dev" -s '*' -a codex --global --yes
```

Writes `~/.codex/skills/` (or `$CODEX_HOME/skills/`). Codex also reads `.codex-plugin/plugin.json` — the `/plugins` flow.

### Amp

```sh
npx -y skills add "jackin-project/jackin-dev" -s '*' -a amp --global --yes
```

Writes `~/.config/agents/skills/`. Amp also offers `amp skill add jackin-project/jackin-dev`.

### OpenCode

```sh
npx -y skills add "jackin-project/jackin-dev" -s '*' -a opencode --global --yes
```

Writes `~/.config/opencode/skills/`. OpenCode also auto-loads the shared `~/.agents/skills/` tree, so a Codex or Kimi-Code global install is visible to OpenCode with no extra step.

### Grok

Grok Build (xAI) isn't yet an agent target in the `skills` CLI, so install the tree by hand. Grok discovers skills from `.grok/skills/` (walked up to the repo root) and `~/.grok/skills/`:

```sh
# global — every repo on this machine
git clone --depth 1 https://github.com/jackin-project/jackin-dev /tmp/jackin-dev &&
  mkdir -p ~/.grok/skills && cp -R /tmp/jackin-dev/skills/* ~/.grok/skills/

# …or per-project
mkdir -p .grok/skills && cp -R /tmp/jackin-dev/skills/* .grok/skills/
```

Verify with `grok inspect`; each skill surfaces as a `/<skill-name>` slash command. Grok also reads `~/.agents/skills/` (AGENTS.md compat) and `.claude/` (Claude Code compat), so if you've already installed there, Grok picks the skills up with no extra step.

### Kimi Code

```sh
npx -y skills add "jackin-project/jackin-dev" -s '*' -a kimi-code-cli --global --yes
```

Kimi Code CLI (the TypeScript agent; `kimi-code-cli`) — writes the shared `~/.agents/skills/` tree.

The [`jackin-the-architect`](https://github.com/jackin-project/jackin-the-architect) role image bakes this in, so every agent it launches has the skills with no per-agent step.

## How it works

**Manual-only** is enforced with `disable-model-invocation: true` frontmatter, with the explicit-invocation `description` as the fallback for agents lacking that field.

**Deferred:** the `plan` skill (break `## Design` → `## Tasks`).

## How skills use `cargo xtask`

The skills shell out to `cargo xtask` subcommands (they live in the jackin' repo and CI reuses them → PR/main parity) for the steps that are **mechanical, exact, or must-not-be-forgotten**. The division is deliberate:

- **The binary** does the deterministic work — scaffold a file, edit sidebar JSON, classify a diff, emit the exact canonical text, run a validator — and hands the agent **facts + a correct skeleton**, then **verifies** the result. It never writes prose or makes a content decision.
- **The agent** does the judgment — read the diff, decide what shipped, write the prose, decide where content moves.

So the binary backstops what an agent slips on (a forgotten verify block, a dangling sidebar entry, a missing schema artifact, broken quoting), and the agent spends its effort on what only it can do.

| Skill | `cargo xtask` used | Binary does (deterministic) | Agent does (judgment) |
|---|---|---|---|
| `propose` | `change new`, `roadmap audit` | scaffold the roadmap `.mdx` + register the sidebar entry; validate the sidebar | pick the group; write Problem / Why It Matters |
| `brainstorm` | — | — | discuss; write `## Design` into the `.mdx` |
| `research` | `research scaffold`, `research check` | create the dossier folder + `meta.json`; validate `pages` against disk | author the brief; write `index` + chapters |
| `create-pr` | `pr body` | change digest + template with verify-locally blocks selected and byte-exact | write Summary / What ships / Behavior changes |
| `merge-pr` | `roadmap retire --plan` / `--apply`, `roadmap audit` (+ CI `schema-check`) | worklist of inbound links + page content; remove the `meta.json` entry, delete the `.mdx`, run the audit, fail on a dangling link | move prose to `guides/`/`reference/`; write the `## Completed` bullet; repoint links |
| `release-check` | — | — | review CI / test / exception results |
| `release-notes` | — | — | classify merged PRs into changelog categories |
| `release` | — | — | version recommendation; confirm; run `cargo release` |

### `create-pr` flow

```
[agent]  branch → commit inline (Conventional Commits, DCO -s) → push
[bin]    cargo xtask pr body --base origin/main
           → classified change digest (rust/docs/capsule/schema + diffstat)
           → PR template with the right verify-locally blocks filled byte-exact
             (Checkout + isolation always; Rust/docs/capsule/schema conditional),
             prose sections left as placeholders
[agent]  fill the prose from the digest + diff
[agent]  gh pr create --body-file  →  reply with URL + verify commands
```

`propose` reuses this `pr body` path for its early PR.

### `merge-pr` flow

```
[agent]  resolve PR; read gh pr view + gh pr diff; blast-radius classify
[bin]    CI gate — gh pr checks (CI already ran schema-check + tests); trust green
   if the PR ships/advances a roadmap item:
[bin]      cargo xtask roadmap retire <slug> --plan
             → worklist: page content, every inbound link, the meta.json entry
[agent]    move operator prose → guides/, design → reference/, write the
           ## Completed bullet, repoint links; commit
[bin]      cargo xtask roadmap retire <slug> --apply
             → remove meta entry, delete .mdx, run audit, FAIL on dangling link
   (partial ship → roadmap retire <slug> --partial sets Status, keeps the page)
[agent]  metadata reconcile
[bin]    squash-merge — jackin-pr-trailers + gh pr merge --squash (title carries (#N))
```

### `schema-check` — a CI gate, not a skill

`cargo xtask schema-check --base origin/main` runs in `.github/workflows/ci.yml`. It detects a `CURRENT_*_VERSION` bump and fails the build unless all required artifacts ship (migration step, `from-<predecessor>/` fixture, `schema-versions.mdx` Timeline entry). No skill owns it — running in CI is what makes the 5-artifact rule enforced rather than reviewer-eyeballed. `merge-pr` only reads its result; the agent may run it locally during `/goal Implement` to self-check before pushing.

### xtask inventory

Built in the jackin' repo and used above: `change new`, `research scaffold`, `research check`, `roadmap audit`, `pr body`, `roadmap retire`, `schema-check`. Full reference: the xtask inventory page in the jackin' docs.

## Requirements

Designed for the `jackin-project/jackin` repository. Expects `gh` authenticated; the release skills additionally expect `cargo-release` installed, a `release.toml` with `pre-release-replacements` for `CHANGELOG.md`, and a `.github/workflows/ci.yml`.

## License

Apache-2.0
