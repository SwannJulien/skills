# Improving spec, plan, build and map — lessons from BookSnap

Findings from using the four skills on BookSnap (`~/Projects/booksnap`,
feature `cycle-de-vie-livre-copie`, US-01 and US-02, 16–19 October 2026), and the
changes they call for. Evidence comes from the Claude Code session transcripts
(`~/.claude/projects/-home-swann-Projects-booksnap/`), the spec and plan
documents, and the commits `beb0921` (US-01), `cbb0e8b` (maps), `f009886`
(US-02).

Everything under **Proposed** is not yet in the skills. The current skills do
what their `SKILL.md` says and nothing more — in particular, `build` neither
reads nor regenerates the maps today.

---

## Part 1 — Context engineering for the pipeline

### What the transcripts show

Files read per session, counting both the Read tool and `cat`/`sed`:

| Session | Files read | Of which the previous stage already read | Final context |
|---|---|---|---|
| plan US-01 | 46 | — | ≈202k tokens |
| build US-01 | 52 | 38 | ≈221k tokens |
| plan US-02 | 42 | — | ≈135k tokens |
| build US-02 | 32 | 24 | ≈89k tokens |

Twenty files were opened in at least three of the four sessions
(`conventions.md`, `GlobalExceptionHandler.java`, `catalog-view.js`, both delete
modals, the repositories…).

Most of that is **not** waste: build must read a file to edit it, and a plan is
only worth something because it was written against the code as it is. The
waste is elsewhere:

- **Re-orientation.** Build explored again instead of trusting the plan's
  references.
- **Re-derived facts.** The US-02 plan rediscovered that returning a copy
  promotes the next pending hold, and that `deleted_at` and `copy.status` are
  independent — the second one already found by the US-01 plan. Both sat in
  "To promote into the spec", which no human ever promoted.
- **The structural memory was unusable.** The maps were stale on arrival (see
  Part 2, §5), so every stage went back to the code.

The goal is not to stop reading files. It is to never derive the same fact twice.

### State of the art (as of mid-2026)

- **Always-loaded instruction files** — `CLAUDE.md`/`AGENTS.md`, Kiro steering
  files, Spec Kit `constitution.md`. Stable rules. We have this:
  `CLAUDE.md` + `.claude/spec/conventions.md`.
- **Memory bank** (Cline): prose files the agent reads at start and rewrites at
  end. Easy, but the prose drifts from the code — nothing ties it to a commit.
- **Context packed into the handoff** (BMAD story files, Spec Kit plan/tasks):
  the earlier stage pays for research once and hands the next stage excerpts,
  references and constraints.
- **Structural indexes**: aider repo map, LSP-backed symbol lookup (Serena),
  semantic indexes. `/map` belongs here, plus cross-layer seams.
- **Agent-managed memory**: Claude Code auto memory, Anthropic memory tool. Good
  for preferences and lessons, weak for code state.
- **Just-in-time retrieval and subagents**: keep pointers, fetch on demand,
  push exploration into subagents that return a condensed brief.

Common lesson: **derived knowledge about code goes stale, and stale memory is
worse than re-reading.** Every layer needs either a freshness check (a commit
sha) or content that doesn't depend on code state (rules, decisions, lessons).

A clean context per stage is a feature — it stops the plan anchoring on the spec
conversation. Memory travels through explicit files, never through shared
conversation.

### Proposed design: four layers

| Layer | File | Holds | Written by | Freshness |
|---|---|---|---|---|
| Project rules | `CLAUDE.md`, `.claude/spec/conventions.md` | how to run, verify, migrate; forbidden commands | human, bootstrap | rarely changes |
| Structure | `docs/map/*` | what exists, what connects to what | `/map` | sha per map; refreshed at end of build |
| Feature memory (**new**) | `<feature>/NOTES.md` | durable facts and traps found while working, with evidence | plan and build append; human prunes | each entry dated, with a code pointer |
| Story handoff | `US-NN-plan.md` | the context packed for one story | plan | disposable after build |

#### 1. Feature `NOTES.md`, appended by agents

Replaces the human-dependent "To promote into the spec" loop. Plan and build
append each durable fact once; every later stage reads it first, right after
the conventions.

Entry shape:

```markdown
- **Returning a copy promotes the oldest pending hold onto it.**
  `BorrowingServiceImpl.returnBorrowing` → `promoteNextPendingHold`.
  Found: plan US-02, 2026-10-19. Affects any walkthrough that returns a copy
  while a hold is pending.
```

Rules: one fact per entry, evidence pointer required, append-only for agents,
the human deletes entries that stop being true. The spec skill reads it too
when writing later stories of the same feature.

#### 2. The plan as a context pack for build

Two new sections in `plan/templates/plan.md`:

- **Read list** — `path:line-range` with one line on why it matters.
- **Established facts** — verified facts with evidence.

`build/SKILL.md` then says: trust these, open only the files you edit, explore
only when a fact proves false — and report it when that happens.

#### 3. Maps kept fresh so stages can use them

Options, to decide:

- **(a)** build ends with `/map` before offering the commit, maps committed with
  the code. Always fresh; costs time and tokens on every story.
- **(b)** build ends with `/map check` and reports which maps are now stale;
  regeneration left to the human before a PR. Cheap; maps may lag.

Either way `build` should also read the relevant maps at the start (it doesn't
today), and `/map` must handle dirty trees (Part 2, §5).

#### 4. Plan exploration in a subagent

The plan's research goes to an `Explore` subagent returning a condensed brief.
Same reading, but the plan's own context stays near 50k instead of 200k.

---

## Part 2 — Other findings, by priority

### 1. End-to-end walkthroughs that can't be performed

- The user had to ask how to create a librarian account (US-01 step 13), how to
  create a hold, and for SQL to set different `end_date`s (US-02 step 1). The
  feature README's shared setup said "set a password via US-007/US-008" — not
  actionable.
- Two US-02 steps were impossible: a return promotes the pending hold, so
  `copyId: null` never appears; step 2's `UPDATE` violates
  `start_date <= end_date`. Found by plan and build, not by spec.

**Proposed**: spec gives exact SQL or commands for every setup state the UI
can't reach, and checks the walkthrough against schema constraints, triggers and
documented business rules. Plan gets a mandatory **Walkthrough feasibility**
section.

### 2. No path for amending a spec after approval

- During US-01 verification the user asked for the refusal message to name the
  students. The agent changed the code, then rewrote US-01's criteria and plan
  to match.
- That absorbed part of US-02, whose spec became wrong ("US-01 already did most
  of the backend work"). The feature README's D9 still says the message is
  generic.
- All six specs were written on 18 October; the Context of US-03 to US-06
  describes code that no longer exists.

**Proposed**: an amendment protocol — a user-requested change updates the spec,
adds a dated line under Decisions, and names the downstream stories it affects.
Plan reads what dependency stories actually shipped (git log, code) and reports
divergence from the spec's Context. Consider `/spec refresh US-NN`.

### 3. Acceptance criteria that prescribe mechanism

- US-01: two of the plan's three spec problems came from mechanism-as-criteria
  (a partial index on `book` "for catalogue reads" that brings nothing; the lint
  wording).
- US-02: criterion 9 "confirmation modals unchanged" contradicted the rest.
- Others named `@EntityGraph`, `@Transactional`, `exception/copy/`.

**Proposed**: tighten the section contract in `spec/SKILL.md` — criteria cover
the contract and the conventions, not the internal mechanism. "No N+1" yes;
"use `@EntityGraph`" is the plan's job.

### 4. Tick discipline only exists inside `/build`

- US-01 build ticked 12/23; another session ticked the remaining 11 on the
  user's word.
- US-02's 9 remaining boxes were ticked **from reading the code** in a session
  with no skill loaded — what `build/SKILL.md` forbids.
- The backend runs from the IDE, so a human always does the final check, and
  the skill has no step for it.
- US-016's code was lost without ever being committed; US-01 was still
  uncommitted after verification.

**Proposed**: `/build verify US-NN` mode that walks the human through the
remaining steps one by one with exact commands and ticks only on an observed
result. A one-line ticking rule in the spec template so any agent sees it. Build
ends by offering a commit.

### 5. Maps stale on arrival

- `/map` ran on uncommitted US-01 code but recorded HEAD `239e023`; the next
  plan rightly treated every map as stale. US-02 build and review never opened
  them.
- `db.md` bakes in live-database state ("V9, not yet applied") — a spurious diff
  on the next run.
- Real drift found and under-reported: `author.name`/`genre.name` are `varchar`
  in the live DB, not `citext` as V4 and `CLAUDE.md` say.

**Proposed**: flag or refuse a run when mapped sources have uncommitted changes;
never write live-system state into a map; surface contradictions with the
project's own docs in the report, not only in Gaps.

### 6. Infrastructure

- **Name clash**: renaming `spec-plan` → `plan` left dangling symlinks and
  `/plan US-01` fell through to built-in plan mode.
- **Conventions drift**: BookSnap's `.spec/conventions.md` still names
  `spec-plan`/`spec-implement` and the flat layout, while the skills hard-code
  `FEAT-<slug>/US-NN-<slug>/`. Make conventions the single source for layout and
  add a format version.
- **Specs untracked**: `docs/implementation` is gitignored in BookSnap — no
  history of amendments or ticks. Bootstrap should detect it and ask.

### 7. Smaller

- The plan prescribed comments; the build over-commented and the user had to
  ask for a trim. Plans never prescribe comments; bootstrap records the
  project's comment policy.
- The lint baseline measured before editing worked well — make it a build rule:
  target is "no new findings", not exit 0.
- The plan missed `ApiAuthorizationMatrixTest`, which asserted the old role.
  Plan step: find tests asserting the behaviour being changed.
- The user reworded the user-facing message twice during verification. Text the
  user reads is written verbatim in the scenarios, reviewed at the spec gate.
- The user's one interview correction was about the UI flow (two modals in
  sequence). For front-end stories, ask about or sketch the interaction flow in
  the proposal.

---

## Backlog

Recommended order. Tick when merged into the skills.

- [ ] Feature `NOTES.md` layer (Part 1, §1) — spec, plan, build
- [ ] Plan as context pack: Read list + Established facts (Part 1, §2) — plan template, build
- [ ] Executable, feasibility-checked walkthroughs (Part 2, §1) — spec, plan
- [ ] Amendment protocol and dependency check (Part 2, §2) — spec, plan, build
  - Done 2026-10-20: build proposes amendments from walkthrough feedback (edit, dated decision, affected stories), applied on approval.
  - Left: plan reads what dependency stories actually shipped; `/spec refresh US-NN`.
- [ ] Criteria cover contract, not mechanism (Part 2, §3) — spec
- [ ] `/build verify` mode, tick rule in template, commit offer (Part 2, §4) — build, spec template
  - Done 2026-10-20: build's "After the user's walkthrough" phase (re-verify every criterion, rerun checks, offer a commit); plan ends with a verdict for users who don't read the plan.
  - Left: tick rule in the spec template, for agents running without the skill.
- [ ] Map freshness: choose (a) or (b), build reads maps (Part 1, §3) — build
- [ ] Dirty-tree handling, no live state, doc contradictions in report (Part 2, §5) — map
- [ ] Conventions as single source for layout, format version, gitignore check (Part 2, §6) — spec bootstrap
- [ ] Plan exploration via subagent (Part 1, §4) — plan
- [x] Automated tests per scenario, tests first (new) — spec, plan, build
  - Done 2026-10-25: bootstrap records test levels (missing harness → enabler story); plan writes a self-contained test map and lists existing tests the story changes (§7 `ApiAuthorizationMatrixTest`); build dispatches a test-writer from spec + test map only, checks red, implements without editing tests, runs the whole suite end-to-end included.
- [ ] Smaller items (Part 2, §7)
- [ ] Optional: `/retro` skill to mine a project's sessions after each feature
