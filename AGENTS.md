# AGENTS.md

A plugin providing development workflow skills for the [jackin](https://github.com/jackin-project/jackin) project.

All skills are **manual-only** — the operator invokes each explicitly as `/jackin-dev:jackin-<name>`. None auto-fire (each `SKILL.md` sets `disable-model-invocation: true`).

## Skills

Eight manual-only skills live under `skills/`: `jackin-propose`, `jackin-brainstorm`, `jackin-research`, `jackin-create-pr`, `jackin-merge-pr` (feature workflow) and `jackin-release-check`, `jackin-release-notes`, `jackin-release`.

- **What each does, the workflow model, and the design** — see [README.md](README.md).
- **The full process for one skill** — see its `skills/<name>/SKILL.md`.

Invoke explicitly as `/jackin-dev:jackin-<name>`; nothing auto-fires.

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
