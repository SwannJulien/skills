---
name: spec
description: Write specifications for a feature as a set of independently verifiable user stories, before any code is planned or written. Use when the user wants to spec out a feature, break a feature into user stories, write acceptance criteria, or start spec-driven development ("spec this out", "write the spec for X", "cut this into user stories", "/spec"). Grills the user on the need decision by decision and pins the domain vocabulary, proposes a decomposition for approval, writes one document per story, then hands over a review digest and records the user's approval. Does not plan implementation and does not write code — those are the plan and build skills.
---

# Spec

First stage of a four-stage pipeline. Each stage hands off through a document, not through context:

| Stage | Skill | Writes | Reads |
|---|---|---|---|
| 1 | `spec` (this one) | `README.md` + `US-NN-*-spec.md`, glossary terms | the repo + the user |
| 2 | `plan` | `US-NN-*-plan.md` | one spec + the repo |
| 3 | `build` | tests, then code | one spec + its plan |
| 4 | `review` | `US-NN-*-review.md` | one spec + its plan + the diff |

Everything stages 2 to 4 need must be **in the documents**. They never see this conversation. A decision you make here that isn't written down will be re-litigated later, or silently reversed.

Your job ends at the specification. **Write no code, plan no implementation, name no file you intend to create.** Describing *what the system does* is the work; describing *how to build it* is stage 2's and it is actively harmful here — it turns the human's review of behavior into a review of code.

## Stage 0 — Conventions

Read `.claude/spec/conventions.md` — where specs live, where the glossary lives, the layers a story can name, how a walkthrough is performed — and the project's agent instructions (`CLAUDE.md` or `.claude/CLAUDE.md`, `AGENTS.md`) for languages, commands and hazards. Then read the glossary, if one exists: its terms are the words the interview and the documents use.

If the conventions file names no glossary, look for `GLOSSARY.md` at the repo root. If there is none, ask where it should go when the first term resolves — once — and record the answer in the conventions file.

If the conventions file is missing, bootstrap it: follow `references/bootstrap.md`, which also checks the agent instructions cover what the three stages need. Write the file, show the user a summary, and ask them to confirm before continuing. This happens once per project. Do not spend interview questions on things that belong in either file.

## Stage 1 — Research before asking anything

Read the code first. Every question you ask that the repo could have answered burns the user's patience and teaches them to skim your output — and skimmed output defeats the human gate this whole pipeline rests on.

**Start from `docs/map/` if it exists.** The `map` skill keeps a generated document per layer — schema, endpoints, components, deployment — where every entity names what it reaches and what reaches it. Reading the relevant maps first tells you in one pass which tables, endpoints and components this feature touches, and it is the fastest way to notice something already exists. Then read the actual code for the parts that matter.

Each map's first line records the commit it was generated at and the globs it covers. **If it is behind on those globs, treat the map as absent and read the code.** A stale map is a confident wrong answer, which is worse than no answer. Never edit a map, and never cite one in a spec in place of the code it describes.

Find out, at minimum:

- Which files, tables, endpoints and components this feature touches, and what they currently do.
- What already exists that the user may not realise exists (half-built, or built and broken).
- The project's own documented rules and hazards (`CLAUDE.md`, `AGENTS.md`, `docs/`, ADRs). These constrain the spec and belong in it. But on what the system is and how it is built — structure, naming, existing behavior — the code is the source of truth: where a document's description disagrees with it, quote the code in Context and report the contradiction to the user. A document's explicit requirements — a *must* or *never*, a stated constraint, a declared migration — still constrain the spec, even where the code falls short of them.
- How the thing is verified today: the test levels and harnesses in `.claude/spec/conventions.md`, the existing tests that cover what this feature touches, and the walkthrough interfaces.

Read the actual code, not just file names. The best specs quote the code that is wrong; the worst paraphrase it from a distance.

## Stage 2 — Grill: walk the decision tree until nothing is assumed

The interview's job is a shared understanding of the need, not just a decomposition. A spec that faithfully describes the solution the user named, when they needed something else, is the most expensive failure in this pipeline: every later stage executes it well, and it surfaces at the walkthrough.

Map the interview as a **decision tree**: every decision branches into the decisions that hang off it. Settle the root first — **the problem**:

- Who has it, and what they do today instead.
- What they will observe once it is solved. This becomes the "so that" of the stories, and Stage 5 checks the scenarios against it; if the user cannot say it, nothing downstream can check it.
- If the request names a solution, what it is for. Arguing back needs the goal, not only the code — the code may offer a cheaper route to the same outcome.

Then the branches the problem opens: roles, scope boundaries, what happens on failure, what is explicitly out, and the words the user uses for things.

**Facts are yours; decisions are theirs.** Anything the repo can answer, look up — in a subagent if your agent can dispatch one, so the interview is not blocked: only the questions that depend on that fact wait for it. Never ask the user what the code says. What the system should do is theirs: put each such decision to them. Implementation choices are neither's here — they belong to plan, so don't ask about them.

### One question at a time

This is an interview, not a questionnaire. Ask **one question**, wait for the answer, and let it choose the next question. A list of ten questions gets ten short answers. One question gets a real answer, and that answer often makes the next three questions pointless.

The **frontier** is every open decision whose prerequisites are settled. Pick the next question from it, closest to the root first: the problem, then what follows from it. Of two questions at the same depth, ask the one whose answer would change more of the tree. After each answer, recompute the frontier. When an answer surprises you, follow it up before moving on.

Questions come in two kinds. Say which kind each one is by how you ask it:

- **Open: the user knows something you don't.** Who has the problem, what they do today, what a good outcome looks like, why they asked for this, what an example case is. You cannot recommend an answer to these, so don't invent one. Ask them to describe or explain, and leave room for a long answer. Most questions about the root are open.
- **Closed: a choice between options you can name.** Give the options and what you found in the code that bears on them, then your recommendation and its reason. Accepting it should take one word, but picking another option or explaining something else should be just as easy. You have read the code, and they are relying on your recommendation.

```
**<title>**: <the question; for a closed one, the options and what the code
says about them>

➡️ <closed only: your recommended answer, and why>
```

If your agent has a structured question tool with selectable options, use it for closed questions and always leave a free-text answer available. Ask open questions in plain text.

**Nothing is silently assumed.** Where you would have assumed, ask a closed question with a recommendation instead. Accepting it costs the user about what reading a written assumption costs, and unlike an assumption buried in one of six documents, the user sees it.

Keep it moving. If the answer to a closed question is obvious from what the user has already said, state it and move on ("you said X, so I'm taking Y; tell me if not"). That is a recorded assumption they can correct, not a silent one. If the user tells you to go with your recommendations for the rest, list what you took and move to Stage 3.

### Sharpen the language as you go

- **Challenge terms.** When the user's word conflicts with the glossary, or with the code's name for the same thing, make it your next question: "the glossary defines *loan* as X; you seem to mean Y — which?"
- **Pin fuzzy words.** "Account" may be the member or the login; propose the precise term.
- **Probe boundaries with concrete scenarios.** Invent the case that forces a choice: "a member with two holds returns a copy — which hold is served?" The answers become the spec's scenarios.
- **Cross-check against the code.** "You said partial cancellation is possible; the code cancels whole orders. Which is right?"

When a term resolves, write it into the glossary **at once**, in the format of `templates/glossary.md` — not in a batch at the end. Create the file lazily, when the first term resolves. Where the code already names the concept, that name is what identifiers keep: if the term the user settles on differs, add the code's name to the entry as _In code_, so plan and build can map one to the other. The glossary holds vocabulary only: no behavior, no decisions, no implementation. Those belong in the spec documents.

### When to stop

When the frontier is empty: every branch visited, every decision answered or put to the user. There is no question budget. But if the tree keeps widening long after the root is settled, the request is probably several features. Say so (see "Arguing back") rather than grilling all of them.

## Stage 3 — Propose the cut, and wait

Output a compact proposal — readable in two minutes — and **stop for approval**. Do not write the story documents yet. This is also where the user confirms you understood them: the first item is the one they should read hardest.

Re-cutting here is free. Re-cutting after you have written twelve documents is not.

The proposal contains, and nothing more:

1. The feature as you now understand it: the problem, who has it, what they will observe once it is solved.
2. The ordered story table: number, title, kind (feature/enabler), scope/layers, one line of objective, what it depends on.
3. For each story with a UI in its scope, a **sketch of the interaction**: the screens or states in order, what the user does and what they see, in plain text or ASCII. Gherkin is hard to picture, and people validate what they can picture. It is not a design — no layout, no styling.
4. What you are deliberately leaving out, and anything you decided without asking — expected to be empty after the grilling, and each entry needs its reason.
5. Anything you think is wrong with the request (see "Arguing back").

### What makes a story a story

**The test is: can it be verified end to end on its own, and shipped without the next story?**

The test is *not* "does it touch every layer". A backend-only story verified through an HTTP collection and the database is a perfectly good story; padding it with UI work to satisfy a layer quota produces fat stories that fail the real test. Record the layers in the `Scope` field and let them be whatever they honestly are.

Two kinds of story are allowed:

- **feature** — the default. User-observable. Gets a user story ("As a … I want … so that …").
- **enabler** — migrations, shared infrastructure, a schema correction. Gets a plain **Objective** instead; forcing a user story onto one produces "As a developer, I want Flyway", which tells the reader nothing. An enabler still needs an observable verification, even if the observation is a database query or a startup failure.

Budget: if more than about one story in four is an enabler, the cut is wrong. Fold the infrastructure into the first story that needs it and re-propose.

One enabler sits outside the budget: **a missing test harness.** If `.claude/spec/conventions.md` marks a test level `missing` and this feature's scenarios need it — any story with a UI or an externally driven interface needs end-to-end — the first story sets that harness up. Its verification is one passing test through it, run by the project's own command. It is paid once per project, not per feature; afterwards update the level in the conventions file.

Order the stories so each is shippable when its turn comes. Prefer a thin end-to-end story first over a "build all the plumbing" story.

### Arguing back

You may push back on the request, and you should when you believe it:

- If it is really one story, propose one story and say why splitting it would produce pieces that can't be verified alone.
- If it is three features wearing one name, say so and propose specs for the one they most likely meant, listing the others.
- If the code shows the premise is wrong — the thing already exists, or is blocked by something they haven't mentioned — lead with that.

Say it once, plainly, with the evidence. If the user reaffirms the original request, that is their decision: build it their way, and record the disagreement under **Decisions and non-goals** so stages 2 and 3 don't reopen it.

## Stage 4 — Write the documents

Only after approval. Write into the location given by `.claude/spec/conventions.md`:

```
<specs-root>/FEAT-<feature-slug>/
  README.md                        # feature overview + ordered story table
  US-01-<slug>/
    US-01-<slug>-spec.md
  US-02-<slug>/
    US-02-<slug>-spec.md
```

The feature folder is always prefixed `FEAT-`. Each story gets **its own folder**, named `US-NN-<slug>`, holding `US-NN-<slug>-spec.md` — stage 2 drops `US-NN-<slug>-plan.md` in beside it, so everything about one story lives in one directory.

Numbering is **per feature folder**, starting at `US-01`. Never global across the project — concurrent features would collide.

Links in the documents are relative: a story links back to its feature as `../README.md`; the feature README links to a story as `US-NN-<slug>/US-NN-<slug>-spec.md`.

Use `templates/spec.md` for each story and `templates/feature-readme.md` for the index. Write in the doc language set in the agent instructions, use the glossary's terms, and include the conventions file's standing acceptance criteria in every story. Every story starts with **Status** `draft`; only Stage 6 changes it.

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
- **Every scenario becomes an automated test**, so the observation must be one a test can make: a status code, a row in a table, a text on the screen, a message on a queue. Write text the user reads verbatim. A judgement only a human can make — "looks right", "feels fast" — is not a scenario; it belongs in the end-to-end verification.
- **Scenarios cover failure and edge cases**, not just the happy path. A spec with only happy paths is a wish.
- **No behavior in the acceptance criteria.** If a criterion would make sense as Given/When/Then, it is a scenario in the wrong section. The criteria are the done-checklist: error types, response codes, where the code lives, "the HTTP collection has a file for this endpoint", "lint passes".
- **The end-to-end verification is derived from the scenarios, not a third copy of them.** It replays them as one sequence through real interfaces — the ones listed in `.claude/spec/conventions.md`, preferring what the system's own user would use. Include the setup (fixtures, accounts) and the cleanup.
- **Write acceptance criteria as `- [ ]` checkboxes.** Stage 3 ticks them; they are the completion signal for the whole pipeline.

## Stage 5 — Check your own work

Before handing back, verify each document:

- [ ] Every scenario is falsifiable by a single observation, and that observation can be made by an automated test.
- [ ] No scenario is restated in the acceptance criteria.
- [ ] Failure and edge cases are covered, not just the happy path.
- [ ] The scenarios together deliver the story's "so that": a reader who saw only these scenarios pass would believe the problem solved. If not, a scenario is missing or the story is mis-stated.
- [ ] Every text the system's user reads — message, label, email — is written verbatim in a scenario.
- [ ] Domain words are the glossary's, and each means the same thing in every document.
- [ ] The story is verifiable alone, without the next one.
- [ ] Context quotes real code or schema, with paths — nothing invented. Re-read anything you are unsure of.
- [ ] Every interview decision and every assumption is recorded in the document, not just in this conversation.
- [ ] No implementation plan, no new code, no invented file paths.
- [ ] The end-to-end verification can actually be performed with this project's tooling.
- [ ] If a test level this feature needs is `missing`, the first story is the enabler that sets it up.

## Stage 6 — Gate 1: hand over a review digest, record the approval

The documents exist; they are not approved. This is the human gate the whole pipeline rests on, and a person handed six documents of a hundred lines each reviews none of them closely. Give them a digest that tells them where to look.

The digest goes in your message, not in a file: it is a view of the documents, and a file would be a second copy that goes stale. It contains:

1. **The files written**, and the story order.
2. **Decisions you made without the user** — gaps found while writing that the interview did not cover — each with a link to the section where it sits. Ideally none; each one is a question you still owe them, so list them here, then ask them one at a time after the digest, with a recommendation, in the Stage 2 format.
3. **Every text the system's user will read**, verbatim, grouped by story. Wording is what most often changes at the walkthrough, where changing it costs a test, the code and an amendment.
4. **One line per story**: its "so that", and how many of its scenarios are failure or edge cases.
5. **What to check hardest**: the places you are least sure the scenarios say what they meant.

Then wait.

- **Corrections**: edit the documents, re-run the Stage 5 checks on what changed, and hand over a digest of the changes only.
- **Approval**: set **Status** to `approved — <YYYY-MM-DD>` in each story the user approved. They may approve some stories and not others; only an approved story can be planned.

The plan skill refuses a story whose status is not `approved`. Never set it on your own judgement — the status records that a human read the spec, and nothing else in the pipeline records that.
