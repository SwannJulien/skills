# Personal AI Skills

In this repository you'll find the skills I have developed and I'm using in my personal projects

## Index

  1. [Installing a skill](#installing-a-skill)
  2. [/add-note](#add-note)
  3. [/line-by-line](#line-by-line)
  4. [/map](#map)
  5. [/spec, /plan, /build, /review](#spec-plan-build-review)

## Installing a skill

Every skill in this repository is a directory containing a `SKILL.md`, plus any
`templates/` and `references/` it reads while it runs. Installing one means
putting that directory — whole — where your agent looks for skills. The
directory name becomes the command, so `spec/` is invoked as `/spec`.

Keep the subdirectories with their `SKILL.md`. A `SKILL.md` copied on its own
will load and then fail to find the files it refers to.

### Globally, for every project

Agents read user-level skills from a directory under your home: `~/.claude/skills/`
for Claude Code, `~/.copilot/skills/` for Copilot, and the same pattern under
whichever dotfile directory your agent uses — check its documentation if you are
unsure. Create it if it doesn't exist, then either copy the skill in:

```bash
cp -r <skill> <agent-skills-dir>/
```

or symlink it, so that pulling this repository updates the installed skill:

```bash
ln -s "$(pwd)/<skill>" "<agent-skills-dir>/<skill>"
```

Symlink while you are still changing the skill — the edit is live on the next
invocation. Copy when you want a frozen version that your edits here won't
disturb.

### In a single project

Put the skill directory in the project's own skills folder — `.claude/skills/`,
`.copilot/skills/`, or your agent's equivalent — and commit it:

```bash
mkdir -p <project>/.claude/skills
cp -r <skill> <project>/.claude/skills/
```

The skill then travels with the repository and behaves identically for everyone
working on it, which is what you want once a team shares a methodology. Prefer
the global install when you are the only user, or when every project should pick
up changes at once.

Both can coexist: a project-level skill takes precedence over a global one with
the same name, so a project can pin its own version.


## /add-note

### Purpose

Turns something you just learned while chatting with an AI (a best practice, a gotcha, a rule of thumb, the difference between two approaches) into a lasting entry in a personal notes file, organized by topic (JS, Java, PostgreSQL, Docker, ...).

Entries are written for a beginner: they explain what the concept is, why it matters, how it works and when to use it, instead of just stating the rule. The result is a set of personal reference files that are easy to reread months later.

### Setup

Install `add-note/` as described in [Installing a skill](#installing-a-skill). The notes themselves are kept in a separate directory of your choice (typically their own repository), not in this repo.

### How it works

Invoke it with a topic and the knowledge to note, e.g. `/add-note JS the difference between == and ===`. If no topic is given, the skill asks for one.

1. **Find the learning directory.** On first use the skill asks where your notes are kept and saves the answer to `~/.config/add-note/config`. Later calls reuse it silently. You can target another directory for a single call by including its path in the request, or ask the skill to change the saved one.
2. **Find the topic file.** The topic maps to `{topic-lowercase}/{topic-lowercase}_notes.md` inside the notes directory (e.g. `javascript/js_notes.md`). If the file doesn't exist, it is created from a standard skeleton with an Index and an Entries section.
3. **Read the whole file** to know the current entry count, existing titles, and whether the notes are flat or already split into chapters.
4. **Check for duplicates.** If a similar entry already exists, the skill asks whether to expand it or add a new one.
5. **Generalize the content.** Project-specific names, business logic, internal URLs and other proprietary details are stripped out so the entry reads as generic guidance.
6. **Write the entry**, numbered sequentially, with:
   - a teacher-like explanation (short paragraphs, bullets, tables or Do/Don't pairs where useful)
   - a small, well-commented code example when it adds clarity
   - a selective line-by-line walkthrough for code a beginner might not follow
   - a Mermaid, table or ASCII diagram for flows, lifecycles or comparisons
   - a **Further reading** link to an authoritative source
7. **File it** at the bottom of the flat list (and the Index), or under the matching chapter if the notes have been reorganized.
8. **Offer to reorganize.** Once the notes reach about 15 entries, or 3+ entries cluster around the same subject, the skill proposes grouping them into chapters, and only does it after you confirm.
9. **Confirm** which entry was added, the full path of the file, and where in it.


## /line-by-line

### Purpose
### How it works


## /map

### Purpose

Keeps a generated map of a codebase — one document per layer, committed — so you
can tell how your own project works after an agent has been writing in it.

The thing you lose when you stop typing the code yourself is not the table list.
It is the **seams**: which endpoint writes which table, which component calls
which endpoint, what is now dead. So every entity in every map names what it
reaches and what reaches it, and that is the column the format exists to force
onto the page.

Run it once when you open a project, and again before each PR. The first run
gives you the shape; every later run gives you a diff.

```
docs/map/
  README.md          index, with each map's commit and whether it is stale
  db.md              tables → columns → types → keys → written by / read by
  api.md             endpoints → auth → contract → handler → tables → called from
  front.md           routes + components → state owned → calls → used by
  deploy.md          environments, pipeline, and where every config value comes from
.claude/map/
  conventions.md     the map set, source globs, introspection commands
```

### Setup

Install `map/` as described in [Installing a skill](#installing-a-skill). The
first run writes `.claude/map/conventions.md` — which maps this project gets, the globs
each is derived from, and the commands that produce a schema or route listing
mechanically. Commit it, along with `docs/map/` itself: the committed maps are
the baseline the next run diffs against.

### How it works

```
/map            first run → proposes a map set, you approve
                → one subagent per map, in parallel
                → a final pass transposes the seams
                → offers a CLAUDE.md pointer

/map            before a PR
                → regenerates only the maps whose sources moved
                → prints a change report, offers it as the PR body

/map api        one map, unconditionally
/map check      staleness only, writes nothing
```

The change report is what you read: added, changed and removed per layer, then
the seams that moved, then a short "worth a look" — an endpoint with no caller, a
new table with no index on the column it is filtered by, a variable the code
reads that nothing sets. The maps are the reference you open when a line in the
report surprises you.

### Design decisions

**A re-run on unchanged code must produce no diff.** This is the constraint the
whole format obeys, and it is enforced structurally rather than hoped for: if a
map's source globs have no commits since the SHA recorded on its first line, the
file is not opened at all. Not regenerated and compared — not touched. Unchanged
code cannot produce a diff, by construction.

**Everything mechanical, sorted, one row per entity.** Tables alphabetically,
endpoints by path, components by file path. No overview paragraph, no
introduction, no conclusion — generated prose re-rolls itself on every run and
buries three real changes in forty reworded ones. The output is tables, because
tables diff.

**One `Why` column, carried over verbatim.** It is the only prose in the output,
and therefore the only thing that can reword itself. It holds what the code
cannot say — why this is denormalised, why this endpoint is deliberately
unauthenticated — and it is empty for most rows. Pinning it across runs means
those notes accumulate instead of being rewritten by whichever model ran last.

**Derive from a mechanical source wherever one exists.** `pg_dump -s`, the ORM
schema, a generated OpenAPI document. Having a model retype 200 column types
from migrations is slow and is the one place it will invent a `varchar(255)` that
was never there.

**Each map owns its outbound edges; inbound edges are transposed, never
re-discovered.** `front` discovers component → endpoint, `api` discovers endpoint
→ table, and a final pass reverses those columns to fill `Called from` and
`Written by`. Two agents grepping for the same relationship from opposite ends
disagree, and a user holding two maps that contradict each other trusts neither.
Dead-code detection then falls out for free: an inbound column that comes out
empty means nobody calls it.

**One subagent per map, in parallel.** Each starts cold with its globs, its
template and its commands, and returns counts and gaps — not narration. The
orchestrator derives the report from `git diff` on the written files, so what you
read is the committed content, not an agent's account of it.

**`?` beats a guess, and every `?` is listed under `Gaps`.** An invented "Used
by" makes you delete live code. The maps say what they could not determine and
why — dynamic dispatch, a caller outside the repo, no access to the database.

**Diagrams are one per map, bounded, and fall back to a table.** A foreign-key
graph, a middleware chain, one route's data flow, the deploy pipeline. Never the
component tree: a 200-node Mermaid graph is unreadable and its diff is garbage.

**The maps are generated-only, and `/map` writes no application code.** Durable
knowledge you want to keep goes in a `Why` cell, which survives, or into your
notes via [`/add-note`](#add-note) — not into a file the next run overwrites. A
run that also "fixed something while it was in there" is a run you can no longer
trust to be read-only.

**`/spec` and `/plan` read the maps, and distrust them when stale.** The pointer
in `CLAUDE.md` means a cold agent gets the project's shape for one read instead
of exploring blind. Both skills check the recorded commit first and treat a map
that is behind on its own sources as absent — a stale map is a confident wrong
answer, which is worse than none.


## /spec, /plan, /build, /review

### Purpose

Spec-driven development in four stages, with two human gates. One skill writes
the specification, a second turns one story into an implementation plan, a
third builds it, a fourth reviews the build before you try it. Project-agnostic.

| Stage | Skill | Writes | Reads |
|---|---|---|---|
| 1 | `spec` | `README.md` + `US-NN-*-spec.md` | the repo + you |
| 2 | `plan` | `US-NN-*-plan.md` | one spec + the repo |
| 3 | `build` | tests, then code | one spec + its plan |
| 4 | `review` | `US-NN-*-review.md` | one spec + its plan + the diff |

One folder per feature, one folder per story inside it, so everything about a
story sits in a single directory:

```
docs/specs/FEAT-multi-library/
  README.md                            # feature overview + ordered story table
  US-01-library-scoped-reads/
    US-01-library-scoped-reads-spec.md
    US-01-library-scoped-reads-plan.md
    US-01-library-scoped-reads-review.md
  US-02-librarian-invitations/
    US-02-librarian-invitations-spec.md
```

Each stage starts with clean context and hands off through a document. Nothing
travels between stages except what is written down — which is why the spec is
required to record its own decisions and the alternatives it rejected.

### Setup

Install all four as described in [Installing a skill](#installing-a-skill);
they are meant to be used together. The first run in a new project writes
`.claude/spec/conventions.md` — where specs live, the layers a story can name,
how a walkthrough is performed, the criteria every story carries. Commit it: all
four skills read it, and it is what keeps project plumbing out of the interview.

Everything else the skills need — languages, build/test/lint commands, how the
app runs, how to reset its state, what must not be run without asking — comes from the
project's `CLAUDE.md`, because any agent doing any task needs it too, not just
these four skills. The first run checks `CLAUDE.md` states the handful of facts
the stages rely on, whatever shape the file has, and proposes only the missing
ones.

### How it works

```
/spec "add multi-library support"
    → reads the code, then grills you in rounds: the problem first,
      each question with a recommended answer "yes" accepts
    → writes resolved terms into the glossary as they settle
    → proposes a cut into stories, with a sketch of each UI flow, and stops
  you: approve or re-cut
    → writes one document per story, status draft
    → hands over a review digest: unasked decisions, every user-facing
      text verbatim, what to check hardest

  you: read the digest and the specs, correct, approve   ← gate 1: the behavior
    → status approved; plan refuses a story that isn't

/plan US-01                 (clean context)
    → writes US-01-…-plan.md beside the spec, in the story's folder
    → ends with "nothing needs you" or what you must decide first

/build US-01                (clean context)
    → a test-writer agent turns each scenario into a test; all fail
    → builds it until they pass, without editing them
    → runs every test level, ticks the criteria, runs what it can of the
      walkthrough and records what each step produced

/review US-01               (clean context)
    → reruns the suite, re-checks every ticked criterion, unticks what doesn't hold
    → reads each scenario's test against the scenario, the diff against the
      spec's decisions, the plan's design choices and the project's rules
    → writes US-01-…-review.md; fixes nothing
    → ends with "ready for your walkthrough" or "back to build"

/build US-01                (answers the review, if anything is blocking)
    → fixes each finding, or disputes it with a reason; /review again

  you: perform the walkthrough, report each point   ← gate 2: the result
    → each point is fixed as a bug, or proposed as a spec amendment
  you: "walkthrough OK"
    → re-verifies every criterion, reruns lint and tests, offers a commit
```

Repeat from `/plan` per story, reading each spec before planning it. Re-run
`/spec` only when the feature changes shape.

There are two human gates. The plan is not one: once you agree on the behavior,
the route is the agent's, and plan's final message tells you if anything needs
you before building. That includes its significant design choices — anything
hard to reverse, a new dependency, a changed contract — each with the
alternative it rejected and what it means for the stories not yet built, which
plan reads so that one story's design doesn't corner the next. A choice worth
remembering comes with a drafted ADR, written only if you accept it.

Each story document carries the same sections: the user story or objective, the
context quoted from the code as it is today, the decisions and non-goals,
Gherkin scenarios for every observable behavior, acceptance criteria as
checkboxes, an end-to-end verification walkthrough, and the traps to watch for.

### Design decisions

**The interview grills the need, not just the cut.** It walks a decision tree
from the root — who has the problem, what they do today, what they will observe
once it is solved — asking every question whose prerequisites are settled in one
round, each with a recommended answer. Facts come from the code, never from you;
decisions come from you, never from a silent assumption. Resolved terms go into
the project's glossary as they settle, so every document and every stage uses
one word for one thing. After Matt Pocock's
[grilling and domain-modeling](https://github.com/mattpocock/skills) skills,
built in rather than invoked so the writing half cannot be skipped.

**Gate 1 is made reviewable, and recorded.** The spec stage ends with a digest
rather than a list of files: the decisions it made without you, every text the
system's user will read, and where it is least sure. Each story carries a
`Status`; only your approval sets it to `approved`, and plan refuses anything
else.

**The code is the source of truth for how things are done here.** Style,
naming, structure and patterns come from the code, not from what `CLAUDE.md`,
ADRs, the glossary, the maps or a plan say about it — descriptions go stale.
When one disagrees with the code, the stages follow the code and report the
disagreement for you to fix the document. Three limits: a document's explicit
requirements — a *must* or *never*, a constraint, a declared migration, what
must not be run — still bind, and code that falls short of one is debt, not an
example; a defect is never a pattern, so a security or correctness problem next
door is reported, not copied; and the behavior a story adds comes from the spec,
because the code doesn't have it yet.

**A story is "independently verifiable and shippable", not "touches every
layer."** A layer quota makes the agent pad backend stories with UI work to
qualify. The story records the layers it honestly touches.

**Two kinds of story.** `feature` gets a user story; `enabler` (migrations,
infrastructure) gets a plain objective, because forcing a user story onto one
yields "As a developer, I want Flyway". Enablers are budgeted at roughly one in
four — more than that means the cut is wrong.

**The cut is approved before the documents are written.** Re-cutting a proposal
is free; re-cutting twelve finished documents is not, and a human asked to review
twelve documents at once reviews none of them. A story with a UI comes with a
plain-text sketch of its flow, because people validate what they can picture.

**One job per section.** Scenarios hold all the behavior. Acceptance criteria
hold everything that isn't behavior. The end-to-end walkthrough replays the
scenarios rather than restating them. When a fact appears twice, one copy goes
stale and the reader can't tell which is normative.

**The plan is a separate, disposable document.** It cites line numbers and goes
stale on first edit; the spec stays true after merge. Keeping them in one file
also turns the behavior review into a code review. Durable knowledge found while
planning gets flagged for promotion into the spec's "Watch out for".

**Checkboxes are the handoff protocol.** `- [ ]` → `- [x]` in the spec is the
only edit the implementer may make to it, and the only mechanical completion
signal in the pipeline.

**Every scenario becomes an automated test, written before the code by a
different agent.** The plan maps each scenario to a test level, a file, and the
entry point it drives. A test-writer agent gets the spec and that map — never
the plan's steps — and writes tests that must fail first. The implementer makes
them pass and may not edit them: a test changed until it passes is a spec
rewritten to match the code. Tests written by the agent that wrote the code
test what it built, not what was asked. Every story with a UI gets an
end-to-end test. The skills prescribe no stack: the first run records the
project's test levels, and a missing harness becomes the first story of the
next feature. The walkthrough stays: the tests prove the code does what the
spec says; the walkthrough is where you find out the spec says what you meant.

**The build is reviewed by an agent that didn't write it, before you try it.**
Build ticks its own checkboxes, and an agent judging its own work judges what it
built — the reason tests come from a separate agent applies to the ticks too.
Review starts cold with the spec, the plan and the diff, never build's reasoning
or report. It reruns the suite, unticks any criterion that doesn't hold, and
checks what tests can't: that each scenario's test asserts every outcome the
scenario names, that no non-goal was built, that design choices the plan didn't
list didn't slip in. It writes findings and fixes nothing — a reviewer that
fixes is a second implementer. Build answers each finding, or disputes it with a
reason; a dispute review still holds goes to you. Review comes before the
walkthrough so your time goes on code already checked against the spec — and it
never replaces the walkthrough, because every agent takes the spec as the truth.

**The spec is written once and amended only by a human.** If implementation shows
the spec is wrong, the stage stops and reports. A spec edited to match the code
is a description, not a specification. When walkthrough feedback changes what
you want, build proposes the amendment — the edit, a dated decision, the later
stories it affects — and applies it only once you approve.

**The spec agent may argue back.** If the request is really one story, or three
features under one name, or already built, it says so with the evidence — once.
If you reaffirm, the disagreement is recorded under "Decisions and non-goals" so
later stages don't reopen it.

