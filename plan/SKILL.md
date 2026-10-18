---
name: plan
description: Write an implementation plan for one already-approved user story specification (a US-NN-*-spec.md document written by the spec skill). Use when the user points at such a spec and wants the implementation worked out before any code is written ("plan US-03", "write the plan for this spec", "/plan"). Second stage of spec-driven development — reads the spec and the repo, writes a sibling plan document, writes no code. Not for general planning, and not a substitute for plan mode.
---

# Plan

Second stage of a three-stage pipeline. Stage 1 (`spec`) produced a specification the human has read and approved. You turn **one** story into an ordered implementation plan. Stage 3 (`build`) executes it.

You write a plan document. **You write no code and you do not modify the spec.**

## Which story

The user names one (`US-03`, a path, "the next one"). If they don't, read the feature `README.md` (one level up from the story folders), find the first story whose acceptance criteria are not all ticked, and confirm that is the one before starting.

Plan exactly one story. Planning two at once produces a plan that only works if both are implemented together, which breaks the one property the spec was cut to have.

## Read, in this order

1. **`.spec/conventions.md`** — how this project builds, verifies, migrates, and what must not be run without asking. If it is missing, the spec skill's bootstrap has not run; say so and stop.
2. **The spec**, whole. Its **Decisions and non-goals** section is binding: a rejected alternative is not yours to re-propose, and a stated non-goal is not yours to include. If you think a decision is wrong, say so to the user — do not quietly plan around it.
3. **The feature `README.md`** — shared setup, feature-wide decisions, ordering.
4. **`docs/map/`**, if it exists — the generated map per layer, which gives you the endpoints, tables and components this story touches and, in its seam columns, what else depends on them. Use it to find what you must read, and to find the callers a change will break. Check each map's first line: if the recorded commit is behind the sources it lists, treat that map as absent. Never cite a map where the plan needs a real path and line, and never edit one.
5. **The code**, properly. Read every file you intend to touch, not just its name. The plan's value is that it is written against the code as it actually is.

The spec is self-sufficient by design — you were never meant to see the conversation that produced it. If something genuinely cannot be resolved from the spec plus the repo, ask the user; don't guess, and don't pad the gap with a plausible invention.

## The plan

Write to `<story-dir>/US-NN-<slug>-plan.md` using `templates/plan.md`, in the language set in `.spec/conventions.md` — beside the spec, inside the story's own folder:

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

### What does not go in the plan

- Behavior. If you find yourself specifying what the system should do, that is the spec's job — and if the spec doesn't say it, that is a gap to report, not to fill.
- Restating the acceptance criteria. Reference them by number.
- The end-to-end verification walkthrough. It is in the spec; the plan does not copy it.

### When the spec is wrong

Reading the code closely is the point at which spec errors surface. If the spec is impossible, self-contradictory, or rests on a false premise about the code:

**Stop and report it.** Do not patch the spec, and do not plan something subtly different from what it says. The human amends the spec, which may change the cut, and you re-plan. That round trip is the methodology working, not a failure.

Keep going for everything that doesn't depend on the problem — deliver the rest of the plan with the gap marked explicitly.

### Knowledge that outlives the plan

The plan is disposable; it starts going stale the moment the first file is edited. But some of what you discover is durable — a trap that will still be a trap in a year, the reason an obvious approach is wrong.

Collect that under **To promote into the spec** at the end of the plan, as a suggestion for the human to move into the spec's **Watch out for** section. You may not edit the spec yourself. Flagging it is how it survives the plan's deletion.

## Finish

Report: the plan path, the step count, any acceptance criterion you could not map to a step, anything you flagged for promotion, and anything in the spec you believe is wrong.
