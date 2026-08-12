# AGENTS.md

A plugin providing development workflow skills for the [jackin](https://github.com/jackin-project/jackin) project.

One `skills/<name>/SKILL.md` source serves every supported native plugin and
Amp's native Claude-plugin compatibility bridge.
Installation, the per-client compatibility matrix, and duplicate-avoidance
rules live in `INSTALL.md`. Never install jackin skills into shared global skill
directories; native plugins are the only supported distribution profile.

All skills are **manual-only**. Claude Code, Grok, and Kimi honor
`disable-model-invocation: true`; Codex uses `agents/openai.yaml` with
`policy.allow_implicit_invocation: false`. Every description begins with an
explicit-request guard for clients that ignore those policy fields.

## Skills

Eleven skills live under `skills/`: `jackin-propose`, `jackin-brainstorm`, `jackin-research`, `jackin-create-pr`, `jackin-refresh-pr`, `jackin-checkout-pr`, `jackin-goal-prompt`, `jackin-merge-pr` (feature workflow) and `jackin-release-check`, `jackin-release-notes`, `jackin-release`. All manual-only.

- **What each does, the workflow model, and the design** — see [README.md](README.md).
- **The full process for one skill** — see its `skills/<name>/SKILL.md`.

Invoke by skill name `jackin-<name>` using the native plugin syntax documented
in `INSTALL.md`.

## Requirements

- The target repository must have `cargo-release` configured with `release.toml`
- The target repository must have a `CHANGELOG.md` with `<!-- next-header -->` marker
- The `gh` CLI must be authenticated
- A `.github/workflows/ci.yml` workflow must exist for CI status checks

## Commit Messages

All commits in this repository MUST follow [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/).

Subject format: `<type>[optional scope][!]: <description>`

Allowed types:

| Type       | Use for                                                |
| ---------- | ------------------------------------------------------ |
| `feat`     | New user-visible feature                               |
| `fix`      | Bug fix                                                |
| `docs`     | Documentation-only change                              |
| `style`    | Formatting, whitespace; no logic change                |
| `refactor` | Internal restructuring; no behavior change             |
| `perf`     | Performance improvement                                |
| `test`     | Adding or updating tests                               |
| `build`    | Build system, tooling, dependencies                    |
| `ci`       | CI configuration                                       |
| `chore`    | Routine maintenance (release, merge, deps)             |
| `revert`   | Reverts a prior commit                                 |

Scope is optional but encouraged when it clarifies the change area.

Breaking changes use `!` after the type/scope (`feat!:` or `feat(api)!:`) and include a `BREAKING CHANGE:` footer in the body.

PR squash-merge: the PR title becomes the commit subject, so PR titles must also follow this convention.
