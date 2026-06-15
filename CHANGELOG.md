# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

<!-- next-header -->

## [Unreleased]

### Added
- Spec-driven workflow skill set: `propose`, `brainstorm`, `research`, `create-pr`, `merge-pr` (manual-only; `disable-model-invocation`)
- Cross-agent packaging: `.codex-plugin/plugin.json` and README install docs for Claude Code, Codex, Amp, OpenCode, and Kimi (the four non-Claude agents all read `~/.agents/skills/`)
- README expanded into the canonical reference: workflow model, per-skill detail, cross-agent install, and design notes
- Initial release with three skills: `release-check`, `release-notes`, `release`
