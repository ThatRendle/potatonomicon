# Potatonomicon — Catalog Orchestration

Your job in this repo is to **orchestrate the research of every potato variety** in the
catalog by delegating to the `potato-researcher` subagent. You are the orchestrator; you
do **not** research varieties yourself.

## The model running this

These instructions are written for **Sonnet**. Stay in the orchestrator role: plan, launch
subagents, track progress, and report. Do not open web searches or fetch sources yourself —
every variety is researched by a `potato-researcher` subagent.

## What already exists

- **`varieties.md`** — the master checklist. **63 unique varieties** across 66 list
  entries, grouped by country (United States, United Kingdom, Australia). This is the
  source of truth for *what* to research and *what's done*.
- **`.claude/agents/potato-researcher.md`** — the subagent definition (runs on Haiku). It
  takes one variety name (+ country), researches it from the web, and writes one
  `<slug>.md` catalog file to the repo root. It owns the file format, the slug rules, and a
  step-0 write-permission preflight. **Do not duplicate or override its rules** — just
  invoke it correctly.
- One `<slug>.md` file per completed variety at the repo root (e.g. `russet-burbank.md`).

## How to invoke the subagent

Use the Agent tool with `subagent_type: potato-researcher`, **one variety per invocation**.
Pass the variety's display name and its country of cultivation, e.g.:

> Research the potato variety "Yukon Gold" (cultivated in the United States). Follow your
> process exactly, including the step-0 preflight write probe. Write `yukon-gold.md` to the
> repo root.

The subagent returns a one-line confirmation (filename written, any `unknown` fields) or a
plain statement that it lacked Write permission. Trust its result; do not re-read every file
to verify unless something looks wrong.

## Starting clean (fresh run)

To reset the catalog and research everything from scratch:

1. **Delete every generated catalog file** — all `<slug>.md` files at the repo root
   **except** `varieties.md` and `CLAUDE.md`. The safe way: remove every `*.md` in the repo
   root, then confirm `varieties.md` and `CLAUDE.md` still exist (they're the two `.md`
   files you must keep). Do **not** touch `.claude/`.
2. **Reset the checklist** — set every variety line in `varieties.md` back to `[ ]` (to do).
   Leave the duplicate-entry annotations as they are.
3. **Run the orchestration loop below** from the top of `varieties.md`.

A line is only "done" when its `<slug>.md` file actually exists at the repo root. If
`varieties.md` and the files on disk ever disagree, trust the files: re-tick from what's
present, or start clean.

## Orchestration loop

1. **Read `varieties.md`** to find the next unticked varieties (`[ ]`). Work in list order,
   top to bottom.
2. **Launch a batch of 4–5** `potato-researcher` subagents in parallel (one variety each).
   Before launching each, mark its line `[~]` (in progress) in `varieties.md`.
3. **Wait for the batch to finish**, then for each result:
   - On success → mark the line `[x]` (done) in `varieties.md`.
   - On failure → see *Write-permission failures* below.
4. **Repeat** with the next batch until every unique variety is done.
5. **Report** a short summary: how many written, which (if any) failed, and the Pontiac
   decision (below).

Keep `varieties.md` as the live source of truth — update it as each file lands, not all at
the end. Legend: `[ ]` to do · `[~]` in progress · `[x]` done.

## Duplicates and merges

- **Cross-country duplicates — one file each.** Three varieties appear in more than one
  country list:
  - Russet Burbank (US + AU)
  - Desiree (UK + AU)
  - Pink Fir Apple (UK + AU)

  Research each **once**, recording the extra country in the file's `also_grown_in` frontmatter.
  **Skip** the duplicate list entries (they're marked as duplicates in `varieties.md`) — do
  not launch a second subagent for them.
- **Pontiac vs Red Pontiac — research separately, do NOT auto-merge.** Australia's *Pontiac*
  and the US *Red Pontiac* may be the same cultivar. Produce a separate file for each
  (`pontiac.md`, `red-pontiac.md`) and **flag this for a human to confirm a merge** in your
  final report. Do not collapse them into one file yourself.

## Write-permission failures

The subagent runs a deliberate step-0 preflight: it tries to write a one-line stub before
researching, and stops immediately if the Write is denied. This can fail intermittently.

- If a subagent reports it lacked Write permission for its file, **re-launch that one
  variety once.**
- If the retry also fails, **record the variety as failed**, leave its `varieties.md` line
  `[ ]`, and **continue with the rest** — do not halt the whole run.
- List every variety that failed both attempts in your final report so a human can fix
  permissions and re-run those.

## Rules

- One variety = one subagent invocation = one `<slug>.md` file at the repo root.
- Never write or edit a `<slug>.md` catalog file yourself — that is the subagent's job. You
  only edit `varieties.md` (the checklist).
- Never fabricate variety data or skip the subagent to "save time."
- Process in batches of 4–5; do not launch all 63 at once.
