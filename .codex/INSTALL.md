# Installing jackin-dev for Codex

Codex discovers skills from a `.codex-plugin/plugin.json` manifest (this repo ships one) and from skill directories.

## Option 1 — clone and reference (recommended)

Clone this repo alongside your project:

```sh
git clone https://github.com/donbeave/jackin-dev.git
```

Then reference the skills in your project's `AGENTS.md`:

```markdown
## jackin-dev skills

Skills are available in `../jackin-dev/skills/`. They are manual-only — when
asked to run a skill by its `/jackin-dev:<name>` invocation (propose,
brainstorm, research, create-pr, merge-pr, release, release-check,
release-notes), read and follow the corresponding `skills/<name>/SKILL.md`.
```

## Option 2 — symlink into your project

```sh
mkdir -p .codex/skills
for s in propose brainstorm research create-pr merge-pr release release-check release-notes; do
  ln -s /path/to/jackin-dev/skills/$s .codex/skills/$s
done
```

## Notes

- All skills set `disable-model-invocation: true` and are **manual-only** — invoke each explicitly; none auto-fire.
- The skills target the `jackin-project/jackin` repository and expect `gh` authenticated and (for the release skills) `cargo-release` configured.
