---
name: build
description: Implement one user story from its approved specification (a US-NN-*-spec.md document) and its plan, then tick off its acceptance criteria and run its end-to-end verification. Use when the user wants a specced and planned story built ("build US-03", "implement this story", "/build"). Third stage of spec-driven development — has a separate agent write failing tests from the spec's scenarios, writes the code that makes them pass without touching them, runs the whole suite end-to-end included, reports which criteria it could not verify, then handles the user's walkthrough feedback and closes the story out when they confirm it ("walkthrough OK", "step 2 of US-03 isn't right"). Not for compiling or running the project, and not for feature work that has no spec.
---

# Build

Third stage of a three-stage pipeline. The specification is approved, the plan is written. A separate agent turns the scenarios into failing tests; you make them pass, verify the rest, and report honestly on what you could not verify.

Two agents on purpose. Tests written by the agent that writes the code test what it built, not what was asked. The test-writer sees the spec and the plan's test map, never the steps, so its tests encode the spec.

## Which story

The user names one. If they don't, read the feature `README.md` and take the first story whose acceptance criteria are not all ticked — then confirm before writing anything.

Each story lives in its own folder under the feature: `FEAT-<feature-slug>/US-NN-<slug>/`, holding `US-NN-<slug>-spec.md` and `US-NN-<slug>-plan.md`. If the plan is missing, stage 2 has not run — say so and stop.

One story per run. The stories were cut to be shippable alone; implementing two at once forfeits that.

## Read before writing

1. **The project's agent instructions** (`CLAUDE.md` or `.claude/CLAUDE.md`, `AGENTS.md`) — build and verify commands, code language, house style, and the commands that must not be run without asking. Respect that last list: it exists because something in this project bites. They outrank the plan on matters of house style, and the plan's author may have missed one.
2. **`.claude/spec/conventions.md`** — where specs live, the test levels, and how a walkthrough is performed.
3. **The spec**, whole. The acceptance criteria are your definition of done. The scenarios are the behavior you owe. **Decisions and non-goals** is binding — a non-goal is not a nice extra to add.
4. **The plan**.

## Tests first

### 1. Dispatch the test-writer

One `general-purpose` subagent, briefed per `references/test-writer.md`: the spec's path, the plan's **Test map** section pasted in, the test levels and commands. Never the plan's path, never the steps.

If your agent cannot dispatch subagents, write the tests yourself before reading the plan's steps, from the spec and the test map alone, under the same rules — and say in your report that the separation was not available.

### 2. Watch them fail

Run every test it wrote or rewrote. Each must fail **because the behavior is missing**: an assertion on the new behavior, a not-found on the endpoint the story adds, a compile error on the symbol a step introduces.

| Result | Meaning | Action |
|---|---|---|
| fails, behavior missing | the test works | go on |
| passes before any code is written | it tests nothing, or the behavior already exists | read it and find out which; a test that tests nothing goes back to the test-writer, existing behavior goes in your report |
| fails for another reason — a broken fixture, a selector into a page that already exists, a misconfigured harness | a test bug | send it back to the test-writer with the output |

Send tests back, don't fix them: continue the same subagent if your agent can, otherwise dispatch a fresh one with the same brief plus the failure. A test you patched is a test the implementer wrote.

End-to-end tests need the app running. Start it as the agent instructions say; if they say to ask first, ask. If it cannot be started, carry on with the other levels and report the end-to-end tests as written but not run — their criteria stay unticked.

## Build it

You are the implementer. **The scenario tests are not yours to change.** They are the spec in executable form, and editing a test until it passes is the same act as rewriting the spec to match what you built. If you believe a test is wrong, decide which kind of wrong before going further:

- **It asserts something the spec does not say**, or contradicts it — a test bug. Send it back to the test-writer, quoting the scenario.
- **It matches the spec, and the spec is what's wrong** — see "When the spec is wrong".

Unit tests for internals the plan creates are yours to add where they help; they don't carry a scenario's name.

Work down the plan's steps in order. Follow the surrounding code's conventions — naming, comment density, error handling, idiom — over your own preferences; a change that reads as foreign is a change the next reader distrusts.

The plan is a route, not a contract. If a step turns out to be wrong or impossible, do the right thing instead and **say so in your report**, with the reason. What you may not do is quietly implement something that doesn't satisfy the acceptance criteria.

Stay inside the story. Problems you notice outside it get reported, not fixed — an unrelated improvement smuggled into the diff is how a reviewable story becomes an unreviewable one.

### When the spec is wrong

If implementation shows the spec is wrong — contradictory, impossible, or resting on a false premise — **stop and report it**. The human amends the spec; the plan may need redoing.

Do not rewrite the spec to match what you built. A spec edited to fit the implementation is no longer a specification, it is a description, and the whole pipeline stops being worth running.

Finish everything that does not depend on the problem, and state plainly what you left out.

## Verify

Two things, in order.

**1. The acceptance criteria.** Go through them one at a time and establish that each actually holds — by running something, not by reading your own diff and feeling confident. Then tick the box in the spec file: `- [ ]` → `- [x]`. The test criterion holds when every scenario has a test carrying its name — search for them — and every one passed red before the code and green after.

That checkbox edit is the **only** change you may make to the spec document on your own initiative. The one other edit is an amendment the user has approved — see "Amending the spec" below.

Leave unticked anything you could not verify, and say why in your report. An unticked box is useful information; a ticked box that isn't true poisons every later stage.

**2. The end-to-end verification walkthrough.** Perform it, in order, as written. Where it needs something you cannot do — a browser you must not install, a service you must not start, a click only a human can make — do every step you can, then stop and tell the user exactly which step they need to perform and what they should see.

Run the project's lint and every level of its tests, end-to-end included, from the agent instructions — the whole suite, not only this story's tests: a story that breaks another story's scenario is not done. If a level the test map needs has no harness, the spec should have opened with an enabler for it; say so, and say which scenarios the verification therefore rests on the walkthrough alone.

## Report

- What you built, and any step where you departed from the plan and why.
- The scenario tests per level, how each failed before the code, any sent back to the test-writer and why.
- Criteria ticked, criteria left unticked and why.
- How far you got through the end-to-end walkthrough, and the exact steps left for the user.
- Lint and test output — including failures, with the output. Never report a story as done over a failing check.
- Anything you noticed and deliberately left alone.
- Anything you believe is wrong in the spec.

State completion plainly when it is true and verified. When it isn't, say which part isn't, without hedging the rest.

## After the user's walkthrough

The report is not the end of the story. The user now performs the walkthrough themselves — it is their acceptance test — and comes back with one of two things: feedback on a step, or a confirmation that it all works.

### Feedback on a step

Before changing anything, say which kind it is — the two need different fixes:

- **A bug** — the code does not do what the spec says, and a scenario's test passed anyway, so that test is too weak. Send the scenario, the user's observation and the test back to the test-writer to strengthen it; watch it fail; then fix the code. Re-verify every criterion the fix could have touched, and untick any that no longer hold.
- **A change of expectation** — the code does what the spec says, and the user now wants something else. Fixing only the code leaves the spec describing behavior that no longer exists, and the next story is planned against it. Treat it as an amendment.

If you can't tell which it is, quote the scenario or criterion and ask.

### Amending the spec

Propose the amendment before applying it, in one message:

1. The exact edit to the spec — the scenario, criterion or walkthrough step as it will read.
2. A dated line to add under **Decisions and non-goals**: what changed and why, in the user's words.
3. The other stories it affects. Read the feature `README.md` and the specs of the stories not yet built; name each one whose Context, scenarios or decisions this change makes wrong, and the feature README's own decisions if they are affected.

Apply it only once the user approves — spec edit, then the test-writer updates that scenario's test and you watch it fail, then the code change, and the README if it is affected. Do **not** edit other stories' specs: list them in your report so the user amends them before planning them.

Never amend the spec to absorb a bug. If the code is wrong, the code changes.

### Walkthrough confirmed

When the user says the walkthrough passed, close the story out — without being asked:

1. Re-verify **every** acceptance criterion, including those you ticked before the walkthrough: code changed since then. Untick any that no longer hold; tick those that were waiting on the user's steps, now that you have their observed result.
2. Run lint and the whole test suite again, end-to-end included.
3. Report: criteria ticked, any left unticked and why, lint and test output, amendments made and the stories they affect.
4. If everything holds, say the story is done and offer to commit it. Never commit without being asked.
