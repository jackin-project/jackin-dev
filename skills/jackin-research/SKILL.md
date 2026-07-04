---
name: jackin-research
description: Produces a standalone multi-page research dossier on the jackin❯ docs site, drawing on web and codebase evidence.
argument-hint: "<slug> [--brief-only] [--in-roadmap] [--web-only|--codebase-only]"
disable-model-invocation: true
---

# jackin-research

Produce a standalone **research dossier** — a multi-page deliverable published on the jackin❯ docs site — for an open question or roadmap topic. Brief-driven: author the brief, then execute it. Gathers and synthesizes evidence (web + codebase); does not make product design decisions.

Reference implementation: `docs/content/docs/research/token-optimization-research/` in the jackin❯ repo.

## When to use

- Operator runs `jackin-research <slug>` for a substantial investigation.

## When NOT to use

- Design decisions on a roadmap item → `jackin-brainstorm`.
- Quick lookup → `deep-research` or Explore directly, no dossier.

## Arguments

- `--brief-only` — author `prompt.mdx` (the brief) and stop; execute later via `/goal Follow <brief>`.
- `--in-roadmap` — store findings inline in the roadmap item instead of a separate dossier (small research only).
- `--web-only` / `--codebase-only` — restrict gathering to one pass.

## Output layout (default: separate folder)

```
docs/content/docs/research/<slug>/
├── meta.json     # { title, defaultOpen:false, pages:[...] } — sidebar order
├── index.mdx     # dossier landing: headline numbers, how-to-read, tier list
├── prompt.mdx    # the brief that was run (the spec; carries the /goal run line)
├── NN-*.mdx      # numbered chapters (00 summary, 01.. foundations, 10-20 areas, 30+ synthesis)
└── tools/        # optional scripts + own meta.json + index.mdx
```
Also add `<slug>` to the parent `docs/content/docs/research/meta.json` `pages`. Big research → own folder (default); small → `--in-roadmap`.

## Process

1. **Scope / brief.** From the topic (or roadmap item), draft or load `prompt.mdx`: mission, chapter list, evidence rules. With `--brief-only`, stop here.
2. **Gather.** Web via the built-in `deep-research`; codebase via Explore/grep. Every external claim carries a source URL; every local number carries its method.
3. **Write.** `index.mdx` (headline numbers + how-to-read + tier list) and the numbered chapters in a fixed per-technique record schema; bundle any reproduction scripts under `tools/`.
4. **Wire the sidebar.** Create/update the dossier `meta.json` and the parent `research/meta.json`.
5. **Docs gate + commit.** Run the `bun` docs gate (`build`, `check:repo-links`, `tsc --noEmit`, `test`) from `docs/`. Commit `docs(research):`; push.

## Common mistakes

- Forcing a Problem/Why/Design shape — a dossier is free-form, not a roadmap item.
- Making design decisions — that is `brainstorm`.
- An unsourced external claim, or a local number without its method.
- `meta.json` `pages` out of sync with the files on disk.

## Tooling

`cargo xtask research scaffold <slug>` creates the folder + `meta.json`; `cargo xtask research check` validates `pages` against disk. Execution typically runs via the external `/goal Follow <brief>`.
