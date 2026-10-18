---
name: build
description: Implement one user story from its approved specification (a US-NN document) and its plan, then tick off its acceptance criteria and run its end-to-end verification. Use when the user wants a specced and planned story built ("build US-03", "implement this story", "/build"). Third stage of spec-driven development — reads the spec and plan, writes the code, reports which criteria it could not verify. Not for compiling or running the project, and not for feature work that has no spec.
---

# Build

Third stage of a three-stage pipeline. The specification is approved, the plan is written. You build it, verify it, and report honestly on what you could not verify.

## Which story

The user names one. If they don't, read the feature `README.md` and take the first story whose acceptance criteria are not all ticked — then confirm before writing anything.

One story per run. The stories were cut to be shippable alone; implementing two at once forfeits that.

## Read before writing

1. **`.spec/conventions.md`** — build and verify commands, code language, and the commands that must not be run without asking. Respect that last list: it exists because something in this project bites.
2. **The spec**, whole. The acceptance criteria are your definition of done. The scenarios are the behavior you owe. **Decisions and non-goals** is binding — a non-goal is not a nice extra to add.
3. **The plan**.
4. **The project's own agent instructions** (`CLAUDE.md`, `AGENTS.md`). They outrank the plan on matters of house style, and the plan's author may have missed one.

## Build it

Work down the plan's steps in order. Follow the surrounding code's conventions — naming, comment density, error handling, idiom — over your own preferences; a change that reads as foreign is a change the next reader distrusts.

The plan is a route, not a contract. If a step turns out to be wrong or impossible, do the right thing instead and **say so in your report**, with the reason. What you may not do is quietly implement something that doesn't satisfy the acceptance criteria.

Stay inside the story. Problems you notice outside it get reported, not fixed — an unrelated improvement smuggled into the diff is how a reviewable story becomes an unreviewable one.

### When the spec is wrong

If implementation shows the spec is wrong — contradictory, impossible, or resting on a false premise — **stop and report it**. The human amends the spec; the plan may need redoing.

Do not rewrite the spec to match what you built. A spec edited to fit the implementation is no longer a specification, it is a description, and the whole pipeline stops being worth running.

Finish everything that does not depend on the problem, and state plainly what you left out.

## Verify

Two things, in order.

**1. The acceptance criteria.** Go through them one at a time and establish that each actually holds — by running something, not by reading your own diff and feeling confident. Then tick the box in the spec file: `- [ ]` → `- [x]`.

That checkbox edit is the **only** change you may make to the spec document. Nothing else in it, not a word.

Leave unticked anything you could not verify, and say why in your report. An unticked box is useful information; a ticked box that isn't true poisons every later stage.

**2. The end-to-end verification walkthrough.** Perform it, in order, as written. Where it needs something you cannot do — a browser you must not install, a service you must not start, a click only a human can make — do every step you can, then stop and tell the user exactly which step they need to perform and what they should see.

Run the project's lint and test commands from `.spec/conventions.md`. If the project has no test suite, the walkthrough is the only real check, and your report should be explicit that that is what the verification rests on.

## Report

- What you built, and any step where you departed from the plan and why.
- Criteria ticked, criteria left unticked and why.
- How far you got through the end-to-end walkthrough, and the exact steps left for the user.
- Lint and test output — including failures, with the output. Never report a story as done over a failing check.
- Anything you noticed and deliberately left alone.
- Anything you believe is wrong in the spec.

State completion plainly when it is true and verified. When it isn't, say which part isn't, without hedging the rest.
