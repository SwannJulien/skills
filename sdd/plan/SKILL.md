---
name: plan
description: Write an implementation plan for one already-approved user story specification (a US-NN-*-spec.md document written by the spec skill). Use when the user points at such a spec and wants the implementation worked out before any code is written ("plan US-03", "write the plan for this spec", "/plan"). Second stage of spec-driven development — reads the spec and the repo, writes a sibling plan document, writes no code. Not for general planning, and not a substitute for plan mode.
---

# Plan

Second stage of a four-stage pipeline. Stage 1 (`spec`) produced a specification the human has read and approved. You turn **one** story into an ordered implementation plan. Stage 3 (`build`) executes it.

You write a plan document — and an ADR, only when the user accepts one you drafted. **You write no code and you do not modify the spec.**

## Which story

The user names one (`US-03`, a path, "the next one"). If they don't, read the feature `README.md` (one level up from the story folders), find the first story whose acceptance criteria are not all ticked, and confirm that is the one before starting.

Check the spec's **Status**. If it is not `approved`, say so and stop: gate 1 has not happened, and planning an unreviewed spec plans whatever the spec agent guessed. The user approves it through the spec skill, or by setting the status themselves.

Plan exactly one story. Planning two at once produces a plan that only works if both are implemented together, which breaks the one property the spec was cut to have.

## Read, in this order

1. **The project's agent instructions** (`CLAUDE.md` or `.claude/CLAUDE.md`, `AGENTS.md`) — how this project builds, verifies, migrates, its languages, and what must not be run without asking.
2. **`.claude/spec/conventions.md`** — where specs live, the layers a story can name, and the test levels: which level proves what, where each lives, which are `missing`. If it is missing, the spec skill's bootstrap has not run; say so and stop. Then the **glossary** it names, if any: write the plan's prose in its terms. Identifiers keep the code's existing names for existing concepts — an entry's _In code_ line maps one to the other — and take the glossary's term only for a concept the code does not name yet.
3. **The spec**, whole. Its **Decisions and non-goals** section is binding: a rejected alternative is not yours to re-propose, and a stated non-goal is not yours to include. If you think a decision is wrong, say so to the user — do not quietly plan around it.
4. **The feature `README.md`** — shared setup, feature-wide decisions, ordering.
5. **The specs of this feature's stories not yet built** — their Scope, Decisions and Scenarios. You plan one story, but its design must not paint the next ones into a corner: a data model a later scenario cannot fit, a contract a later story must break. Read them to judge this story's choices, never to plan their work.
6. **`docs/map/`**, if it exists — the generated map per layer, which gives you the endpoints, tables and components this story touches and, in its seam columns, what else depends on them. Use it to find what you must read, and to find the callers a change will break. Check each map's first line: if the recorded commit is behind the sources it lists, treat that map as absent. Never cite a map where the plan needs a real path and line, and never edit one.
7. **The code**, properly. Read every file you intend to touch, not just its name. The plan's value is that it is written against the code as it actually is.
8. **The tests** that cover what this story changes — search them for the endpoints, components, functions and messages the spec's scenarios change. A test asserting the old behavior will fail once the story is built, correctly; the plan must say so before it surprises anyone. Read the helpers and fixtures the existing tests use, too: the new tests should use them.

**The code is the source of truth for how things are done here** — coding style, naming, structure, patterns. The agent instructions, ADRs, the glossary, the maps and the conventions file mostly *describe* the code, and descriptions go stale: where a description disagrees with the code, plan to the code and report the disagreement, so the human corrects the document. Three limits:

- **A document's explicit requirements still bind** — a *must* or *never*, a stated constraint, a declared migration ("new code uses B"), what must not be run. The code cannot contradict a requirement, only fall short of it: code that does is debt to report, not an example to follow.
- **A defect is never a pattern.** If the nearest pattern is a security or correctness problem — a query built by concatenation, a swallowed exception — don't copy it: plan the correct version and report the existing one.
- **The behavior this story adds comes from the spec**, because the code does not have it yet.

Where the code disagrees with itself, follow the pattern nearest to what this story changes — unless a declared migration names the other one — and say which you chose.

The spec is self-sufficient by design — you were never meant to see the conversation that produced it. If something genuinely cannot be resolved from the spec plus the repo, ask the user; don't guess, and don't pad the gap with a plausible invention.

## The plan

Write to `<story-dir>/US-NN-<slug>-plan.md` using `templates/plan.md`, in the doc language set in the agent instructions — beside the spec, inside the story's own folder:

```
<specs-root>/FEAT-<feature-slug>/US-NN-<slug>/
  US-NN-<slug>-spec.md     # stage 1, yours to read, never to edit
  US-NN-<slug>-plan.md     # this document
```

What makes it useful:

- **Ordered steps, each one leaving the project in a state that compiles.** The implementer works down the list; a step that only makes sense after step 7 belongs after step 7.
- **Real paths, real identifiers.** `UserService.findByEmail` at a path, not "the user service". If you name a file, you have read it.
- **Code only where prose would be ambiguous** — a signature, a query, a config line. A step that reproduces the whole final file is not a plan, it is an implementation you are asking someone to copy.
- **Every acceptance criterion is reachable.** Before finishing, walk the spec's criteria and confirm each one is satisfied by some step. Map them in the trace table. A criterion with no step is a hole in the plan; a step serving no criterion is either scope creep or a missing criterion — say which.
- **Name the step where it will break.** You have read the code; you know where the trigger fires at the wrong time, where the filter intercepts before the controller, where the flush has to happen for the exception to be catchable. That knowledge is the most valuable thing in the document. Put it on the step it belongs to.

### Design choices

The spec settled the behavior; the design is yours — but some design choices are the user's to know about, because they outlive the story. List under **Design choices** each choice where a reasonable engineer could have gone the other way **and** at least one of these holds:

- **Hard to reverse** — a schema shape, a data migration, a persisted or exported format, an identifier others will store.
- **Adds a dependency** — a library, a service, a new piece of infrastructure.
- **Changes a contract outside the story** — a public endpoint, an event, a component or function other code already uses.
- **Departs from the project's established pattern** — the obvious reading of the codebase would assume the opposite.

For each one: the choice, the alternative you rejected and why, what reversing it would cost, and what it means for the stories not yet built — the one it serves, the one it constrains. A choice that follows the existing pattern and is cheap to undo is not listed; most plans have zero to two entries. A list of ten is a sign you are listing everything and the user will read none of it.

Where a choice conflicts with a later story's scenarios or decisions, say so in the entry. If no design satisfies both, that is a problem in the cut: report it under **Problems found in the spec** — you do not choose which story loses.

**ADRs.** A choice that is hard to reverse, surprising without context, *and* the result of a real trade-off deserves a decision record that outlives the plan. Draft it in the entry — one to three sentences: context, decision, why. Don't write the file yourself: offer it in the report. If the user accepts, write it into the project's existing ADR directory (`docs/adr/`, `docs/decisions/`, or whatever the repo uses), numbered after the highest existing one; if there is none, propose `docs/adr/` first. Most choices fail at least one of the three tests, and that is fine — the plan still records them for this story.

### The test map

Build hands this section, and only this section, to a separate test-writer agent that reads the spec but never the plan's steps. It writes the scenario tests before any code exists, so they encode the spec rather than the route. Write the section for that reader: **complete on its own**, with no "see step 3".

For each scenario in the spec, one row:

- **Level** — the cheapest level that can make the scenario's observation through the interface the scenario names. What the system's user observes from outside is end-to-end; anything lower goes to the smallest level in `.claude/spec/conventions.md` that can see it. Use those levels, nothing else.
- **Test file** — a real path, following the project's existing layout and naming.
- **Drives** — the entry point the test calls: an endpoint and method, a route and the elements it uses, a public function's signature. Real identifiers. Where a step creates the entry point, give it exactly as the step will create it; the test-writer codes against that and the test fails until the step lands.
- **Setup** — the fixtures, accounts and state it starts from, naming existing helpers where they exist.

Every story with a UI in its Scope gets at least one end-to-end test along its main path, whatever the levels of its other scenarios.

Then two lists:

- **Existing tests this story changes** — each test that asserts behavior the spec changes, what it asserts today, and the scenario that supersedes it. The test-writer rewrites them; the implementer never touches them.
- **Not automated** — a scenario no test can observe with this project's harness, and why. It is left to the walkthrough. This should be rare; often it means the scenario is not falsifiable and the spec needs amending — say which.

The steps never write the scenario tests; that is the test-writer's job, from this map. A step may still add test infrastructure the map needs and the project lacks — a fixture loader, a page object — if the map names it.

### What does not go in the plan

- Behavior. If you find yourself specifying what the system should do, that is the spec's job — and if the spec doesn't say it, that is a gap to report, not to fill.
- Restating the acceptance criteria. Reference them by number.
- The end-to-end verification walkthrough. It is in the spec; the plan does not copy it.
- Test code. The test map says what each test drives and from what state; the test-writer decides how it asserts.

### When the spec is wrong

Reading the code closely is the point at which spec errors surface. If the spec is impossible, self-contradictory, or rests on a false premise about the code:

**Stop and report it.** Do not patch the spec, and do not plan something subtly different from what it says. The human amends the spec, which may change the cut, and you re-plan. That round trip is the methodology working, not a failure.

Keep going for everything that doesn't depend on the problem — deliver the rest of the plan with the gap marked explicitly.

### Knowledge that outlives the plan

The plan is disposable; it starts going stale the moment the first file is edited. But some of what you discover is durable — a trap that will still be a trap in a year, the reason an obvious approach is wrong.

Collect that under **To promote into the spec** at the end of the plan, as a suggestion for the human to move into the spec's **Watch out for** section. You may not edit the spec yourself. Flagging it is how it survives the plan's deletion.

## Finish

Report: the plan path, the step count, the test count per level, the existing tests this story will change, any scenario left unautomated and why, any acceptance criterion you could not map to a step, the design choices and what they mean for later stories, anything you flagged for promotion, and anything in the spec you believe is wrong.

The user may never open the plan — agreeing on the spec is their gate, the route is yours. So anything that needs them must be in this report, not only in the document. End with exactly one of:

- **Nothing needs you — run `/build`.** Only when no criterion is unmapped, no scenario is left unautomated, nothing in the spec is wrong, and there are no design choices.
- **Before building, you need to:** a short numbered list. Each item is a spec amendment, or a decision the user must make, with why building without it would go wrong. Put each design choice to them as a question with your recommendation, worded so "yes" accepts it — and, where you drafted an ADR, whether to record it. Most are a "yes"; the point is that they are seen before the code makes them expensive.

Items for promotion into **Watch out for** never block the build. List them under the verdict as optional.
