---
name: review
description: Review one built user story (a US-NN-*-spec.md document whose build has run) against its specification and plan, from a clean context, before the user's walkthrough. Use when the user wants a story's build checked ("review US-03", "check what build did", "/review"). Fourth stage of spec-driven development — reads the spec, the plan and the diff, never the build conversation; checks every ticked criterion by running something, checks each scenario's test asserts what the scenario says, checks the diff against the spec's decisions, the plan's design choices and the project's rules; unticks what does not hold, writes a sibling review document, and fixes nothing. Not for reviewing code that has no spec, and not a substitute for the user's walkthrough.
---

# Review

Fourth stage of a four-stage pipeline. Build has made the scenario tests pass, ticked the criteria it could verify, and reported. You check that work from outside it, before the user spends time on the walkthrough.

A fresh agent on purpose. Build ticks its own boxes, and an agent that wrote the code judges what it built, not what was asked — the same reason build has a separate test-writer. You see the spec, the plan and the code, never build's reasoning or its report, so what you find is what the documents and the code show.

Your job is to find, not to fix. **You change no code, no test, no plan.** The one edit you may make to the spec is unticking a box you have shown to be false — never ticking one. Everything else goes into the review document, and build applies it — except a durable fact about the code, which you append to the feature's notes like any other stage (see "Write the review").

The review checks the code against the spec. It does not check the spec against what the user wanted: that is the walkthrough, and nothing here replaces it.

## Which story

The user names one. If they don't, read the feature `README.md` and take the first story that has a plan, ticked criteria and no review document, or a review document with open findings — then confirm before starting.

Each story lives in `FEATNN_<feature-slug>/US-NN-<slug>/`, beside `US-NN-<slug>-spec.md` and `US-NN-<slug>-plan.md`. You write `US-NN-<slug>-review.md` there. If the plan is missing, or no criterion is ticked and no scenario test exists, build has not run — say so and stop.

## What the story changed

Build never commits unasked, so the story is usually the uncommitted working tree: the diff against `HEAD`, plus untracked files. If the tree is clean, the story was committed — ask the user for the base commit rather than guessing.

Attribute every changed file to the story or not, from the spec's **Scope**, the plan's steps and its test map. A change that belongs to neither is either scope creep or someone else's work in progress; if you can't tell which, list it and ask. Never review someone else's work as part of the story.

## Read before judging

1. **The project's agent instructions** (`CLAUDE.md` or `.claude/CLAUDE.md`, `AGENTS.md`) — lint and test commands, how the app runs, house rules, and the commands that must not be run without asking. Respect that last list.
2. **`.claude/spec/conventions.md`** — the test levels, and the glossary it names.
3. **The feature's `NOTES.md`**, beside its `README.md`, if it exists — facts and traps earlier stages found in this code, each with a pointer.
4. **The spec**, whole. Its criteria, scenarios and **Decisions and non-goals** are what you judge against.
5. **The plan** — its **Design choices**, test map and steps. It tells you what build was meant to do, so you can see where it didn't and whether it said so.
6. **The diff**, and around every hunk, enough of the file to judge it against the code next to it.
7. **The previous review document**, if one exists — see "A second round".

## Check

Establish each point by running or reading something specific, and record the evidence. "Looks fine" is not evidence.

### 1. The ticked criteria

Every ticked criterion, one at a time: does it hold now? Establish it the way build should have — by running something, not by reading the diff and agreeing with it. The test criterion holds when every scenario has a test carrying `US-NN: <scenario title>` — search for them.

A criterion that does not hold: untick it in the spec, and record why as a blocking finding. A criterion left unticked: check build's stated reason holds, if the spec or the code makes it visible; don't tick it.

### 2. The scenario tests

For every scenario, read its test against the scenario's `Given`, `When`, `Then` and `And` lines:

- **Does it assert every `Then` and `And`?** A test that passes without checking an outcome lets a bug through to the walkthrough. This is the most valuable thing you check.
- **Does it drive the entry point the test map names**, through its real interface — or does it stub what it is meant to prove?
- **Does it assert anything the spec doesn't say** — a private function, a CSS class, an incidental order?

A weak test is a blocking finding against the test, not the code: build sends it back to the test-writer.

### 3. The code against the spec

- Behavior the spec does not ask for, and above all anything listed as a non-goal or rejected under **Decisions and non-goals**.
- A scenario's outcome the code produces only on the path its test happens to take — the failure and edge cases next to it.
- Every user-facing text against the spec's verbatim wording.

### 4. The code against the plan

- A step done differently, or not done. Departures are allowed — the plan is a route — but you can't see build's report, so record each one with what the diff shows; the user compares it with what build said.
- A design choice made in the code that the plan's **Design choices** doesn't list — hard to reverse, a new dependency, a contract changed outside the story, a departure from the project's pattern. Those were meant to reach the user before the code made them expensive.
- An existing test the plan said this story changes, left as it was — or one it didn't list, changed anyway.

### 5. The code against the project

The code is the source of truth for how things are done here. Judge the diff against the code next to it — naming, structure, error handling, idiom — and against the agent instructions where they *require* something. A description that disagrees with the code is a note for the human, not a finding against the story.

Look hardest for what the tests can't see: security and correctness defects, an error swallowed, a query built from input, a migration that can't be undone, a command the agent instructions say must not be run. A defect copied from the code next door is still a defect. So is walking into a trap the feature's `NOTES.md` records.

If your agent has a code-review command, you may run it on the story's files as an extra pass; its findings go through the same classification below.

### 6. The whole suite

Run lint and every test level, end-to-end included, as the agent instructions say — once, whatever build reported. Start the app if the end-to-end level needs it; if the instructions say to ask first, ask. Record the output. A failing check is blocking, even in another story's test.

## Classify every finding

| Kind | Meaning | Goes to |
|---|---|---|
| **Blocking** | a criterion that doesn't hold, a test that doesn't prove its scenario, a non-goal built, a defect, a binding rule broken, a failing check | build, before the walkthrough |
| **Should fix** | a convention not followed, an unlisted design choice, scope creep, an unreported departure | build, or the user waves it through |
| **Note** | a document that disagrees with the code, a problem outside the story, something you deliberately left alone | the user |
| **Spec problem** | the code does what the spec says and the spec looks wrong | the user, as a possible amendment — never fixed here |

Each finding gets a number, the file and line, what is wrong, the evidence, and the spec line, plan entry or rule it is judged against. Don't propose the fix beyond a sentence: the route is build's.

Hold the line on what counts. A preference is not a finding; "I would have written it differently" is not a finding. A short review with three real findings is worth more than a long one the user skims.

## Write the review

Use `templates/review.md`, in the doc language set in the agent instructions. It records the diff it reviewed — base commit, and whether the tree was dirty — the criteria you unticked, the suite output, and the findings by kind.

A durable fact about the code you established while checking — one a later story would otherwise rediscover — goes in the feature's `NOTES.md`, not the review, which is about this story. Append it in the file's format, `Found: review US-NN, <YYYY-MM-DD>`, one fact per entry with its pointer. Read the file first and don't repeat an entry; never edit or delete one — an entry the code contradicts is a note for the user.

## A second round

When a review document already exists, build has answered it. Check its findings first: each one is **resolved** (show the evidence), **still open**, or **disputed by build** — read build's reason in the document and judge it; a dispute you still hold goes to the user, not back to build a second time. Then review what changed since, as above. Rewrite the document: previous findings with their status, then the new ones.

## Report

- The review path, and the diff you reviewed.
- Criteria you unticked, and why.
- The findings, blocking first, one line each.
- Lint and test output, including failures.
- Changes you could not attribute to the story.
- The entries you appended to `NOTES.md`.

End with exactly one of:

- **Ready for your walkthrough.** Only when nothing is blocking. List any should-fix findings for the user to send to build or wave through, then point at the spec's **End-to-end verification** section: the user performs it, or reads the observed result of every step build recorded and judges each against what they meant.
- **Back to build:** the blocking findings, numbered. The user runs `/build US-NN` on the review — in build's session or a new one — then `/review US-NN` again.
