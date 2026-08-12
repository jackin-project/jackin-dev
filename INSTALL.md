# Installing jackin-dev with native plugins

This is the cross-agent installation contract for jackin-dev. One shared
`skills/` tree is packaged natively for Claude Code, Codex CLI, Grok Build,
Kimi Code, and Antigravity CLI.

## One profile only

Use native plugins exclusively. Do not copy or symlink jackin skills into
`~/.agents/skills/`, `~/.claude/skills/`, `~/.config/opencode/skills/`,
`~/.kimi-code/skills/`, or `~/.gemini/config/skills/`. Plugin plus directory
copies produce duplicate skills and may bypass manual-only policy.

OpenCode and Amp do not have a compatible native repository-plugin channel for
this package. They are intentionally unsupported rather than installed through
a shared global skills directory.

## Claude Code

```sh
claude plugin marketplace add jackin-project/jackin-dev
claude plugin install jackin-dev@jackin-dev
```

Pin the marketplace source to `jackin-project/jackin-dev@v0.4.0` for a
production install. Update with `claude plugin update jackin-dev@jackin-dev`.
Invoke `/jackin-dev:jackin-<name>`.

## Codex CLI

```sh
codex plugin marketplace add jackin-project/jackin-dev
codex plugin add jackin-dev
```

Pin the marketplace source to `jackin-project/jackin-dev@v0.4.0` when required.
Refresh with `codex plugin marketplace upgrade jackin-dev`, then reinstall or
enable the current plugin snapshot. Invoke `$jackin-<name>` or use `/skills`.
Every skill has `policy.allow_implicit_invocation: false`.

## Grok Build

```sh
grok plugin install jackin-project/jackin-dev@v0.4.0 --trust
```

If Claude Code already provides the same plugin through its compatibility
registry, do not install a second Grok copy. Invoke `/jackin-<name>`.

## Kimi Code

```text
/plugins install https://github.com/jackin-project/jackin-dev/tree/v0.4.0
/plugins reload
```

Invoke `/skill:jackin-<name>`. Do not retain any user-skill copy after moving
to the managed plugin.

## Antigravity CLI

```sh
git clone --depth 1 --branch v0.4.0 https://github.com/jackin-project/jackin-dev.git
agy plugin install ./jackin-dev
```

Invoke `/jackin-<name>`. Do not also copy skills into Gemini or Antigravity
skill directories.

## Upgrade and duplicate audit

At each release, update all native plugin channels to the same latest tag and
verify that these commands return no jackin directory copies:

```sh
find ~/.agents/skills ~/.claude/skills ~/.config/opencode/skills \
  ~/.kimi-code/skills ~/.gemini/config/skills \
  -maxdepth 1 -name 'jackin-*' -print 2>/dev/null
```

Then inspect the native managers (`claude plugin list`, `codex plugin list`,
`grok plugin list`, Kimi `/plugins`, and `agy plugin list`). Exactly one native
plugin source should be active per client.
