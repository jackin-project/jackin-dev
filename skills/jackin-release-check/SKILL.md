---
name: jackin-release-check
description: Runs the jackin❯ release-readiness gates.
argument-hint: "[context]"
disable-model-invocation: true
---

# jackin-release-check

Run the release-readiness **gates**. Each gate is binary — green passes, red blocks — and the release is ready only when every gate is green (warnings surface but do not block). The commands are the repo's own (from `CONTRIBUTING.md` merge-readiness + `.github/workflows/ci.yml`), never stripped-down substitutes.

## When to use

- Before cutting a release, or when the operator asks whether the project is release-ready.

## When NOT to use

- Cutting the release itself → `jackin-release` (it runs this skill as its first gate).

## Process

Run each gate in order; collect a green / red / skip verdict into a one-line-per-gate report.

1. **CI gate.** `gh run list --workflow=ci.yml --branch=main --limit=1 --json conclusion --jq '.[0].conclusion'` — green only on `success`. If `docker/construct/**` or `docs/**` changed since the last tag (`git diff --name-only <last-tag>..HEAD -- <path>`), also require `construct.yml` / `docs.yml` green on main.

   *Done when* every CI workflow that owns a changed path is green.

2. **Static gates.** `cargo fmt --check`; `cargo clippy --all-targets --all-features -- -D warnings`. Red on any violation.

   *Done when* both exit 0.

3. **Test gate.** `cargo nextest run --all-features`; `cargo nextest run -p jackin --features e2e --profile docker-e2e`. Red on any failure.

   *Done when* both suites pass.

4. **Doc-link gate.** Walk internal links under `docs/content/docs/` (relative paths + `href`s); verify each target file exists. Red only on a broken link to a shipped page; warn on external URLs you cannot reach.

   *Done when* every internal link resolves.

5. **TODO freshness.** Read `TODO.md` and `todo/`; flag items referencing completed work or showing no recent activity. Warn — does not block.

   *Done when* every TODO item is either live or flagged stale.

6. **Security exceptions.** Read `SECURITY_EXCEPTIONS.md`; ask the operator whether each entry is still current. Review (blocks) until confirmed or the file is updated and re-read.

   *Done when* the operator confirms the catalog.

7. **Direct-commit audit.** `git log <last-tag>..HEAD --oneline --no-merges`; for each commit, `gh pr list --state merged --search "<sha>"`. Warn on commits with no PR — does not block.

   *Done when* every non-merge commit since the last tag is matched to a PR or listed as ungrouped.

## Report

One line per gate: `✓` green, `✗` red (block), `⚠` warn, `-` skip. A red gate stops the release; a warn surfaces for the operator to accept or fix.

## Common mistakes

- Running `cargo test --locked` or `cargo clippy -- -D warnings` (stripped flags) — those are not the repo's gates; the real commands carry `--all-targets --all-features` and use `nextest`.
- Treating a warn as a block, or a red gate as a warn.
- Skipping the path-specific CI workflows when `docker/` or `docs/` changed since the last tag.
- Inventing a doc-link check instead of walking the real `docs/content/docs/` tree.
