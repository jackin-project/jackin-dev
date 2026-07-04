# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

<!-- next-header -->

## [Unreleased]

## [0.3.0] - 2026-07-04

### Changed
- **BREAKING:** prefixed every skill with `jackin-` (`propose` → `jackin-propose`, …, `release` → `jackin-release`) so they no longer collide in the shared `~/.agents/skills/` tree. Skill names are now `jackin-<name>` everywhere.
- Made `SKILL.md` descriptions agent-neutral triggers (per the [agentskills.io](https://agentskills.io/specification) spec: describe what + when, not how to invoke). Dropped the Claude-Code-only `/jackin-dev:` syntax from descriptions and bodies; the README now documents per-agent invocation.
- Rebranded the stylized project mark `jackin'` → `jackin❯` across docs.
- README install section now uses each agent's native CLI where one exists (`claude plugin`, `codex plugin`, `amp skill add`, `grok plugin install`); OpenCode and Kimi Code fall back to the `skills` CLI. All six verified locally.

### Added
- Spec-driven workflow skill set: `jackin-propose`, `jackin-brainstorm`, `jackin-research`, `jackin-create-pr`, `jackin-merge-pr` (manual-only; `disable-model-invocation`)
- Cross-agent packaging: `.codex-plugin/plugin.json` and per-agent README install docs for Claude Code, Codex, Amp, OpenCode, Grok, and Kimi Code
- README expanded into the canonical reference: workflow model, per-skill detail, cross-agent install, and design notes; each skill documents its `cargo xtask` usage (`pr body`, `roadmap retire`, `schema-check`)
- Initial release with three skills: `jackin-release-check`, `jackin-release-notes`, `jackin-release`

### Fixed
- All eight skills now set `disable-model-invocation: true` (the three release skills were missing it; docs already claimed manual-only).
- Removed the unused `hooks/hooks.json` scaffold (`{"hooks": {}}`, unreferenced by either manifest).
- Plugin manifest `homepage`/`repository` URLs (Claude + Codex) point at `jackin-project/jackin-dev` (were the personal `donbeave/jackin-dev`)
- `jackin-release-check` scans `docs/content/docs/` (was the pre-Fumadocs `docs/src/content/docs/`, which validated nothing)
