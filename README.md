# jackin-dev

Development workflow plugin for the [jackin](https://github.com/jackin-project/jackin) project. Provides skills for the spec-driven feature workflow, pull requests, research, release management, validation, and changelog generation.

All skills are **manual-only** — invoke each explicitly as `/jackin-dev:<name>`; none auto-fire.

## Skills

### Feature workflow

| Skill | Description |
|---|---|
| `propose` | Open a feature/idea as a roadmap item draft + early PR (never codes) |
| `brainstorm` | Freeform design discussion → writes the item's `## Design` |
| `research` | Standalone multi-page research dossier on the docs site (web + codebase) |
| `create-pr` | Open a small-fix PR with the correct body + auto-selected verify blocks |
| `merge-pr` | Pre-merge gate + roadmap retirement + squash-merge |

The feature path is `propose → brainstorm → /goal Implement <slug>.md → merge-pr`. Small fixes use `create-pr → merge-pr`. See [SKILLS_PROPOSAL.md](SKILLS_PROPOSAL.md) for the full design.

### Release

| Skill | Description |
|---|---|
| `release-check` | Pre-release validation: CI status, tests, docs, TODOs, security exceptions |
| `release-notes` | Generate changelog from merged PRs in Keep a Changelog format |
| `release` | Full release orchestrator: check → notes → version → cargo release |

## Installation

### Claude Code

```
/plugin marketplace add jackin-project/jackin-marketplace
/plugin install jackin-dev@jackin-marketplace
```

### Codex

See [.codex/INSTALL.md](.codex/INSTALL.md).

### Amp Code

Amp reads `AGENTS.md` and discovers skills from `.claude/skills/` compatibility paths. Point Amp at this repo or symlink the skills directory.

## Requirements

These skills are designed for the `jackin-project/jackin` repository and expect:

- `cargo-release` installed (`cargo install cargo-release`)
- `gh` CLI authenticated (`gh auth login`)
- `release.toml` configured with `pre-release-replacements` for CHANGELOG.md
- `.github/workflows/ci.yml` running tests, clippy, and fmt on every push/PR

## License

Apache-2.0
