---
name: map
description: Generate and maintain a map of a codebase — one document per layer (database schema, API surface, front-end components, deployment) as deterministic committed markdown — then report how the shape of the system changed since the last run. Use the first time you open a project ("map this project", "document how this all works", "/map") and before raising a pull request to see what moved ("what changed structurally", "update the maps", "/map before I push"). Writes reference documents only: it never changes application code, writes no specifications and plans no work.
---

# Map

Code an agent wrote is code you did not type. The first thing that goes is not the table list — it is the **seams**: which endpoint writes which table, which component calls which endpoint, what is now dead. A map is a derived document per layer that forces those seams onto the page.

Two properties do all the work. Protect them over everything else:

- **Mechanical, sorted, one row per entity.** A re-run on unchanged code produces *no diff*. That is what makes the diff you do get worth reading.
- **Seam columns.** Every entity names what it reaches and what reaches it.

The user reads the change report. The maps are the reference they consult when the report surprises them.

## Modes

| Invocation | Situation | What you do |
|---|---|---|
| `/map`, no `.claude/map/` yet | first run in a project | bootstrap conventions, get approval, generate every map, offer the `CLAUDE.md` pointer |
| `/map` | before a PR | regenerate only the maps whose sources moved, re-transpose the seams, report the delta, offer the PR body |
| `/map db`, `/map api front` | after manual surgery, or a map you distrust | regenerate the named maps unconditionally |
| `/map check` | anytime | report staleness per map and stop — **write nothing** |

## Stage 0 — Conventions

Read `.claude/map/conventions.md` from the repo root.

If it is missing, bootstrap it: follow `references/bootstrap.md`, write the file, show the user the proposed map set, and **wait for approval** before generating anything. This happens once per project. A wrong map set discovered after four full generations is an expensive thing to fix.

## Stage 1 — Decide what to regenerate

Every map carries its provenance on line 1:

```
<!-- map: db | sha: 5036f95 | sources: prisma/schema.prisma, prisma/migrations/** -->
```

For each map in the conventions file:

```bash
git diff --name-only <recorded-sha>..HEAD -- <its globs>
```

| Result | Action |
|---|---|
| no files | **do not open the file.** Not regenerated, not compared, not touched |
| files changed | regenerate |
| map file absent | generate |
| recorded sha not in history (rebase, squash) | regenerate |

Rule 1 of this skill: **unchanged sources cannot produce a diff, by construction.** Do not re-derive a map "to check" — hoping two generations agree is exactly the non-determinism the format exists to avoid.

One carve-out, and only this one: the seam pass in Stage 3 may edit the *inbound* columns of a map whose own sources did not change. That is correct — a new caller in the front end genuinely changes who calls an endpoint.

Report what you decided before dispatching: which maps regenerate, which are current, and the commit range. In `check` mode, stop here.

## Stage 2 — One subagent per map, in parallel

Dispatch all regenerating maps in a single message, one `general-purpose` subagent each. They are independent and context-hungry; running them in your own context would blow it on files the user never needs to see.

Each subagent's brief must contain all of this — it starts cold and sees none of this conversation:

1. **The map name**, its source globs, and the introspection commands from `.claude/map/conventions.md` (`pg_dump -s`, an OpenAPI dump, a route-listing command).
2. **The template to follow**, by absolute path: `templates/<map>.md`, or `templates/generic.md` for a project-specific map. The output must match its structure, section order and column order exactly.
3. **The output path**, and the previous version's path if one exists.
4. **The determinism rules** and **the honesty rules** below, verbatim or by reference.
5. **Leave inbound seam columns empty.** Stage 3 fills them. Say which columns those are for its map.
6. **What to return**: entity counts per section, the gaps it recorded, and the mechanical sources it used. Nothing else — no prose summary, no narration of what changed. You derive the report from the files in Stage 4, not from what an agent says about them.

### Determinism rules, for every map

- **Prefer a mechanical source over reading code.** `pg_dump -s`, the ORM schema file, a generated OpenAPI document, a framework route listing. Deriving 200 column types from migrations by hand is slow, and it is the one place a model will invent a `varchar(255)` that isn't there.
- **Sort everything by a stable key** — tables and enums alphabetically, endpoints by path then method, components by file path, config variables alphabetically. Never by file-system or discovery order.
- **Fixed vocabularies only** in classification columns. The templates define them. "container" and "smart component" must not be the same thing written two ways.
- **Carry `Why` cells over verbatim** for entities that have not changed. The `Why` column is the only prose in the output and therefore the only thing that can reword itself; pinning it is what lets it accumulate value across runs instead of being rewritten by whichever model ran last. A `Why` is for what the code cannot say — why this is denormalised, why this endpoint is versioned. Leave it empty otherwise, which is most of the time.
- **No overview paragraph, no introduction, no conclusion.** They get re-rolled on every run and the user already knows what their project does.

### Honesty rules, for every map

- **`?` beats a guess.** An invented "Used by" makes the user delete live code. An honest `?` makes them look.
- **Every `?` and every omission goes in the map's `Gaps` section**, with the reason — not enough coverage, dynamic dispatch, an endpoint called from a client outside this repo.
- **Name real paths and real identifiers.** `routes/invitations.ts:14`, not "the invitations route".
- **Record what exists, not what should.** A map is not a review. Note a dead endpoint under its empty inbound column; do not propose deleting it.

## Stage 3 — Transpose the seams

Each map owns the **outbound** edges of its own entities:

| Map | Owns (discovered) | Receives (transposed) |
|---|---|---|
| `front` | component → endpoint (`Calls`) | component → component (`Used by`) |
| `api` | endpoint → table (`Tables`) | `Called from`, reversed out of `front.Calls` |
| `db` | — | `Written by` / `Read by`, reversed out of `api.Tables` |
| `deploy` | workflow → env, var → consumer | — |

**Inbound columns are the transpose of an outbound column, never a second discovery.** Two agents grepping for the same relationship from opposite ends will disagree, and a user holding two maps that contradict each other trusts neither.

Dispatch one final `general-purpose` subagent for this. It reads every map in `docs/map/` — including the ones that were not regenerated — reverses the owning columns, and fills the inbound ones. It is a mechanical pass: it may not invent an edge that no outbound column asserts. Where a map's outbound source doesn't exist in this project (an API with no front end in the repo), it writes `external` rather than leaving the column blank, so an empty cell keeps meaning "nobody".

An inbound column that comes out empty is a dead-code candidate. The transpose finds these for free; list them in the map's own "Unreferenced" section and carry them into the report.

## Stage 4 — The change report

Derive it by diffing the written files against the recorded SHA:

```bash
git diff --stat <recorded-sha>..HEAD -- docs/map/
git diff <recorded-sha>..HEAD -- docs/map/
```

Write it with `templates/change-report.md`. It is short by contract: one screen, grouped by layer, `+` added, `~` changed, `-` removed. Then the part that justifies the whole skill:

- **Seams that moved** — a new endpoint and who calls it, a table that gained a writer, state that changed owner.
- **Worth a look** — an endpoint with no caller, a component used nowhere, a new table with no write path, a new endpoint with no test. Facts with an obvious question attached, not advice.
- **Maps left untouched**, named. Knowing the front end did not move is information.

Print it. Then **offer** to put it in the PR description — `gh pr edit <n> --body` — and do that only if the user says yes. It is an outward-facing edit to a shared artifact; never do it unprompted, and never on a first run when there is no PR.

On a first run there is no delta. Report the shape instead: entity counts per map, the biggest gaps, and anything already unreferenced.

## Stage 5 — Wiring, first run only

**Offer** a `CLAUDE.md` pointer, so any agent opening this project gets its shape for one read. Only with approval — it is their instruction file, not yours:

```markdown
## Project shape

`docs/map/` holds a generated map per layer — schema, API surface, components,
deployment. Read the relevant one before exploring. Each file states the commit
it was generated at; if that is behind on the sources it lists, trust the code.
Regenerate with `/map`. Never hand-edit a map.
```

Then tell the user to commit `docs/map/` and `.claude/map/conventions.md`. The maps are committed on purpose: they are the baseline the next run diffs against, and the context a cold agent reads. If they would rather gitignore them, say plainly that the pre-PR diff stops working and the skill loses most of its point.

## The rules the orchestrator keeps

- **You write no application code.** Not a fix, not a rename, not a tidy-up, however obvious. A map run that also edits the project is a map run the user cannot trust to be read-only.
- **Maps are generated-only.** If you find hand-written prose in one, do not silently destroy it: tell the user which file and which section, and that durable knowledge belongs in a `Why` cell (which survives) or in their notes (`/add-note`) — not in a file the next run overwrites.
- **Ask before running anything that touches a live system.** Introspection commands come from `.claude/map/conventions.md`; if one needs a database that must be started, or credentials, ask rather than start it.
- **Split a map that outgrows one file.** Past roughly 250 rows, `api.md` becomes `api/` with a `README.md` index plus one file per domain. Record the split and its grouping in `.claude/map/conventions.md` so it is stable on the next run; a grouping that is re-decided every time is a diff that is never readable.

## Finish

Report: which maps were regenerated and which were not, the entity counts, every gap and `?` the subagents recorded, the dead-code candidates, and whether the report went into a PR. If a subagent could not reach a mechanical source — the database was down, the OpenAPI dump failed — say so plainly and say which map is therefore weaker than it looks.
