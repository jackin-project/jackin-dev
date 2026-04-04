# jackin-dev

Development workflow plugin for the [jackin](https://github.com/donbeave/jackin) project. Provides skills for release management, validation, and changelog generation.

## Skills

| Skill | Description |
|---|---|
| `release-check` | Pre-release validation: CI status, tests, docs, TODOs, security exceptions |
| `release-notes` | Generate changelog from merged PRs in Keep a Changelog format |
| `release` | Full release orchestrator: check → notes → version → cargo release |

## Installation

### Claude Code

```sh
claude plugin add /path/to/jackin-dev
```

Or add to your project's `.claude/settings.json`:

```json
{
  "plugins": ["/path/to/jackin-dev"]
}
```

### Codex

See [.codex/INSTALL.md](.codex/INSTALL.md).

### Amp Code

Amp reads `AGENTS.md` and discovers skills from `.claude/skills/` compatibility paths. Point Amp at this repo or symlink the skills directory.

## Requirements

These skills are designed for the `donbeave/jackin` repository and expect:

- `cargo-release` installed (`cargo install cargo-release`)
- `gh` CLI authenticated (`gh auth login`)
- `release.toml` configured with `pre-release-replacements` for CHANGELOG.md
- `.github/workflows/ci.yml` running tests, clippy, and fmt on every push/PR

## License

Apache-2.0
