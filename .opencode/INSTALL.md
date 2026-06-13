# Installing jackin-dev for OpenCode

OpenCode discovers Agent Skills from skill directories. jackin-dev ships plain
`SKILL.md` files (no JS plugin), so point OpenCode at this repo's `skills/`
tree.

## Prerequisites

- [OpenCode](https://opencode.ai) installed

## Install

Clone this repo, then add its `skills/` directory to OpenCode's skill paths in
`opencode.json` (global `~/.config/opencode/opencode.json` or project-level):

```json
{
  "skills": {
    "paths": ["/path/to/jackin-dev/skills"]
  }
}
```

Alternatively, symlink the skills into OpenCode's project skill directory
(`.opencode/skill/`, singular):

```sh
mkdir -p .opencode/skill
for s in propose brainstorm research create-pr merge-pr release release-check release-notes; do
  ln -s /path/to/jackin-dev/skills/$s .opencode/skill/$s
done
```

Restart OpenCode.

## Usage

Use OpenCode's native `skill` tool:

```
use skill tool to list skills
use skill tool to load propose
```

All jackin-dev skills are **manual-only** — load each explicitly; none auto-fire.

## Tool mapping

These skills are written in the portable SKILL.md format. Where a skill names a
Claude Code tool, map to OpenCode's equivalent:

- `Task` with subagents → `@mention` syntax
- `Skill` tool → OpenCode's native `skill` tool
- File operations → your native tools
