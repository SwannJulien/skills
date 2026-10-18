---
name: spec
description: Write specifications for a feature as a set of independently verifiable user stories, before any code is planned or written. Use when the user wants to spec out a feature, break a feature into user stories, write acceptance criteria, or start spec-driven development ("spec this out", "write the spec for X", "cut this into user stories", "/spec"). Interviews the user, proposes a decomposition for approval, then writes one document per story. Does not plan implementation and does not write code — those are the plan and build skills.
---

# Spec

First stage of a three-stage pipeline. Each stage hands off through a document, not through context:

| Stage | Skill | Writes | Reads |
|---|---|---|---|
| 1 | `spec` (this one) | `README.md` + `US-NN-*.md` | the repo + the user |
| 2 | `plan` | `US-NN-*-plan.md` | one spec + the repo |
| 3 | `build` | code | one spec + its plan |

Everything stage 2 and 3 need must be **in the documents**. They never see this conversation. A decision you make here that isn't written down will be re-litigated later, or silently reversed.

Your job ends at the specification. **Write no code, plan no implementation, name no file you intend to create.** Describing *what the system does* is the work; describing *how to build it* is stage 2's and it is actively harmful here — it turns the human's review of behavior into a review of code.

## Stage 0 — Conventions

Read `.spec/conventions.md` from the repo root.

If it is missing, bootstrap it: follow `references/bootstrap.md`, write the file, show the user a summary, and ask them to confirm before continuing. This happens once per project. Do not spend interview questions on things that belong in this file.

## Stage 1 — Research before asking anything

Read the code first. Every question you ask that the repo could have answered burns the user's patience and teaches them to skim your output — and skimmed output defeats the human gate this whole pipeline rests on.

Find out, at minimum:

- Which files, tables, endpoints and components this feature touches, and what they currently do.
- What already exists that the user may not realise exists (half-built, or built and broken).
- The project's own documented rules and hazards (`CLAUDE.md`, `AGENTS.md`, `docs/`, ADRs). These constrain the spec and belong in it.
- How the thing is verified today (per `.spec/conventions.md`).

Read the actual code, not just file names. The best specs quote the code that is wrong; the worst paraphrase it from a distance.

## Stage 2 — Interview: at most 10 questions

**Spend questions only where different answers produce different stories.** Scope boundaries, which users or roles are involved, what happens on failure, what is explicitly out, what "done" means for the user. Not implementation choices — those are not yours to decide or to ask about.

For everything else: **assume, and write the assumption down.** A stated assumption the user can correct in review is cheaper for them than a question they have to answer up front. This typically halves the interview.

Mechanics:

- Use `AskUserQuestion`, maximum 4 per call, two or three rounds at most.
- Give a recommended option first where you have a view. You have read the code; they are relying on that.
- Ten is a ceiling, not a target. Three good questions is a better interview than ten adequate ones.
- Stop early once nothing left would change the decomposition.

## Stage 3 — Propose the cut, and wait

Output a compact proposal — readable in two minutes — and **stop for approval**. Do not write the story documents yet.

Re-cutting here is free. Re-cutting after you have written twelve documents is not.

The proposal contains, and nothing more:

1. One paragraph on the feature as you now understand it.
2. The ordered story table: number, title, kind (feature/enabler), scope/layers, one line of objective, what it depends on.
3. What you are deliberately leaving out, and assumptions you made instead of asking.
4. Anything you think is wrong with the request (see "Arguing back").

### What makes a story a story

**The test is: can it be verified end to end on its own, and shipped without the next story?**

The test is *not* "does it touch every layer". A backend-only story verified through an HTTP collection and the database is a perfectly good story; padding it with UI work to satisfy a layer quota produces fat stories that fail the real test. Record the layers in the `Scope` field and let them be whatever they honestly are.

Two kinds of story are allowed:

- **feature** — the default. User-observable. Gets a user story ("As a … I want … so that …").
- **enabler** — migrations, shared infrastructure, a schema correction. Gets a plain **Objective** instead; forcing a user story onto one produces "As a developer, I want Flyway", which tells the reader nothing. An enabler still needs an observable verification, even if the observation is a database query or a startup failure.

Budget: if more than about one story in four is an enabler, the cut is wrong. Fold the infrastructure into the first story that needs it and re-propose.

Order the stories so each is shippable when its turn comes. Prefer a thin end-to-end story first over a "build all the plumbing" story.

### Arguing back

You may push back on the request, and you should when you believe it:

- If it is really one story, propose one story and say why splitting it would produce pieces that can't be verified alone.
- If it is three features wearing one name, say so and propose specs for the one they most likely meant, listing the others.
- If the code shows the premise is wrong — the thing already exists, or is blocked by something they haven't mentioned — lead with that.

Say it once, plainly, with the evidence. If the user reaffirms the original request, that is their decision: build it their way, and record the disagreement under **Decisions and non-goals** so stages 2 and 3 don't reopen it.

## Stage 4 — Write the documents

Only after approval. Write into the location given by `.spec/conventions.md`:

```
<specs-root>/<feature-slug>/
  README.md              # feature overview + ordered story table
  US-01-<slug>.md
  US-02-<slug>.md
```

Numbering is **per feature folder**, starting at `US-01`. Never global across the project — concurrent features would collide.

Use `templates/spec.md` for each story and `templates/feature-readme.md` for the index. Write in the language set in `.spec/conventions.md`.

### The section contract

Each section has exactly one job. The whole value of this format is that nothing is said twice, because the day a fact appears in two sections is the day one of them goes stale and the reader can't tell which is normative.

| Section | Holds | Never holds |
|---|---|---|
| **Story** / **Objective** | why this exists, in one sentence of intent | behavior detail |
| **Context** | what exists today, quoted from the code; the rules that constrain this | what should change |
| **Decisions and non-goals** | choices made and alternatives rejected, with the reason; what is out of scope | behavior |
| **Scenarios** | every externally observable behavior, one per decision — the *only* place behavior is described | anything untestable |
| **Acceptance criteria** | everything that is *not* behavior: contract shape, error codes, naming, where things live, lint/tests passing | restatements of scenarios |
| **End-to-end verification** | one ordered walkthrough a human can actually perform, with its setup | new behavior claims |
| **Watch out for** | traps, the tempting wrong fix and why it's wrong, hazards from the project's own docs | general advice |

Rules that follow from it:

- **A scenario must be falsifiable by a single observation.** "The system is secure" is not a scenario. "A librarian from another library gets 403" is.
- **Scenarios cover failure and edge cases**, not just the happy path. A spec with only happy paths is a wish.
- **No behavior in the acceptance criteria.** If a criterion would make sense as Given/When/Then, it is a scenario in the wrong section. The criteria are the done-checklist: error types, response codes, where the code lives, "the HTTP collection has a file for this endpoint", "lint passes".
- **The end-to-end verification is derived from the scenarios, not a third copy of them.** It replays them as one sequence through real interfaces — the UI where there is one, otherwise the project's HTTP collection and database shell. Include the setup (fixtures, accounts) and the cleanup.
- **Write acceptance criteria as `- [ ]` checkboxes.** Stage 3 ticks them; they are the completion signal for the whole pipeline.

## Stage 5 — Check your own work

Before handing back, verify each document:

- [ ] Every scenario is falsifiable by a single observation.
- [ ] No scenario is restated in the acceptance criteria.
- [ ] Failure and edge cases are covered, not just the happy path.
- [ ] The story is verifiable alone, without the next one.
- [ ] Context quotes real code or schema, with paths — nothing invented. Re-read anything you are unsure of.
- [ ] Every interview decision and every assumption is recorded in the document, not just in this conversation.
- [ ] No implementation plan, no new code, no invented file paths.
- [ ] The end-to-end verification can actually be performed with this project's tooling.

Then report to the user: the files written, the story order, and what you need them to check hardest — the cut itself, and any assumption you made in place of a question.
