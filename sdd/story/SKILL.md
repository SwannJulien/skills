---
name: story
description: Take one approved user story (a US-NN-*-spec.md document) from plan to "ready for your walkthrough" in a single session — plan, build, review and the fixes the review asks for — stopping only where the user's judgment is needed. Use when the user wants a specced story carried through without launching each stage themselves ("/story US-03", "run US-03 end to end", "take US-03 to the walkthrough", "carry on with US-03"). Orchestrates the plan, build and review skills and adds no rule of its own — plan and review run in subagents with a clean context, build runs in this session. Not for writing the spec (the spec skill), and not for work that has no spec.
---

# Story

The plan, build and review skills each end by telling the user which command to run next. Between the spec's approval and the walkthrough, that makes the user a scheduler: nothing in those handoffs needs their judgment, only their keyboard. You do the scheduling instead.

**This skill adds no rule.** Each stage's own `SKILL.md` stays the authority for that stage; follow it whole. The one thing you override is its last line: where a stage says "run `/build`" or "run `/review` in a clean context", you run it and carry on.

One story per run, like every stage.

## Who runs what

| Stage | Runs in | Model | Why |
|---|---|---|---|
| `plan` | a subagent | the most capable | starts cold from the spec; its exploration of the code stays out of this context |
| `build` | this session | this session's | it is the implementer, it dispatches the test-writer, and the user's walkthrough feedback comes back here |
| `review` | a fresh subagent, every round | the most capable | it must never see what build did or why |

The clean context each stage was designed around is kept: a subagent sees its brief and the files, nothing of this conversation.

**The models differ on purpose.** Working out a route through code as it is, and judging a diff against a spec, are reasoning work; following a good plan against failing tests is not, and a lighter model does it well. So this skill is meant to be launched in a session running the model the user builds with, and it dispatches plan and review on the most capable model the agent offers — on Claude Code, `model: opus` on the dispatch, from a Sonnet session. A reviewer on a different model from the implementer is also a little less likely to share its blind spots. The test-writer stays on the session's model, as the build skill dispatches it.

If your agent cannot choose a subagent's model, say so in your first line — plan and review will run on the session's model, and the user may prefer to run them themselves.

If your agent cannot dispatch subagents, this skill has nothing to offer: say so, and point the user at `/plan`, `/build` and `/review` in separate sessions.

## Where to start — the documents are the state

The user names a story. If they don't, read the feature `README.md`, take the first story whose acceptance criteria are not all ticked, and confirm it before starting.

Then read the story's folder and start at the first row that matches. Say in one line what you found and where you start.

| What the folder shows | Start at |
|---|---|
| Spec **Status** is not `approved` | stop — gate 1 has not happened; that is the spec skill's |
| No plan | 1. Plan |
| A plan; no scenario test, no criterion ticked | 2. Build — if the plan lists **Design choices** or **Problems found in the spec** that you did not put to the user in this session, ask first whether they were settled |
| Criteria ticked, no review document | 3. Review |
| A review document with `open` findings | 3. Review, at "Back to build" |
| A review document whose findings are all `fixed` or `disputed`, verdict still "back to build" | 3. Review, next round |
| A review document with verdict "ready for walkthrough" | 4. Hand over |

## Briefing a subagent

A brief holds exactly three things:

1. **The skill to follow** — by name, and by the absolute path of its `SKILL.md`, which sits beside this skill's folder (`../plan/SKILL.md`, `../review/SKILL.md`), in case the subagent cannot load skills by name. It follows that file whole.
2. **The story** — the absolute path of its spec.
3. **This rule, verbatim**: *You cannot reach the user. Where the skill tells you to ask, confirm or wait for them, do everything that does not depend on the answer, then return the question in your final report with your recommendation. Your final report is the skill's own report, unshortened.*

Nothing else. No summary of the spec, no opinion on the story — and for review, nothing about the build: not its report, not what was hard, not where to look. A reviewer told where to look reviews what the implementer was worried about.

## 1. Plan

Dispatch the plan subagent. Its report ends with one of two verdicts.

**"Nothing needs you"** — tell the user in one line, with the plan's path, and go straight to build.

**"Before building, you need to"** — the interview is yours to run, since the subagent cannot: show the numbered titles, then ask the items one at a time, in the order and format of the plan skill's "One question at a time". After each answer:

- **A choice accepted** — nothing to do.
- **A choice rejected, or an ADR accepted** — send the decision to the plan subagent to revise the plan or write the ADR: continue the same one if your agent can, otherwise a fresh one with the brief plus the decision. Don't edit the plan yourself; you are about to become its implementer.
- **A spec problem** — stop. Amending the spec is the user's, through the spec skill or by hand. Once it is amended, re-dispatch plan.

When every item is answered and no amendment is outstanding, go to build.

## 2. Build

Load the build skill and follow it from "Read before writing" to "Report" — in this session, test-writer subagent included.

Give the user build's report, then carry on to review instead of ending the turn. Stop here only if:

- **The spec is wrong** — build's "When the spec is wrong".
- **A command needs the user's leave** — the agent instructions say to ask first.
- **A check fails and you cannot make it pass** — review would only return it as blocking. Report it with the output.

Criteria left unticked because a step needs a human are not a reason to stop; they wait for the walkthrough.

## 3. Review

Dispatch a **fresh** review subagent — never continue the previous round's. A second round learns what happened to the first one's findings from the review document, as the review skill's "A second round" intends, not from memory of having written them.

If its report carries a question — a changed file it cannot attribute to the story, a base commit it needs — put it to the user, then dispatch again with the answer added to the brief as a fourth item.

**"Back to build"** — follow the build skill's "After the review": answer every blocking and should-fix finding, fill each **Status**, rerun lint and the suite. Then review again, with a fresh subagent.

At most **two rounds** on your own. If the second review still has a blocking finding, or still holds a dispute, stop and give the user the findings: something here needs a judgment neither agent can make.

**"Ready for your walkthrough"** — answer its should-fix findings as "After the review" says, rerun lint and the suite, and hand over. One exception: a should-fix that is a design choice the plan did not list is the user's to decide, not yours to fix — carry it into the handover as a question.

## 4. Hand over for the walkthrough

This is the message the user reads; everything before it was progress notes. It contains:

1. **What ran** — one line: plan, build, how many review rounds.
2. **The walkthrough** — a link to the spec's **End-to-end verification**; each step with what build observed when it performed it; the steps only the user can perform, and what they should see.
3. **Criteria left unticked**, each with its reason.
4. **What needs their eye** — build's departures from the plan; the review's findings and what became of each (`fixed`, `disputed`); its notes and spec problems, which are theirs; any design choice still to decide.
5. **Lint and tests** — the result of the last run, failures with their output.
6. **Entries appended to `NOTES.md`**, by any stage.

Then wait. The user's answer — feedback on a step, or "walkthrough OK" — is handled by the build skill's "After the user's walkthrough", in this session, as written.

## The only stops

- The spec is not approved, or turns out to be wrong.
- Plan has a design choice, an ADR or a spec problem to put to the user.
- A command the agent instructions say must be asked for.
- A check that cannot be made to pass.
- A review question about what belongs to the story.
- A second review round that still blocks.
- The walkthrough.

Anything else gets a one-line progress note at each stage change, and you carry on. A stop not on this list is the scheduling this skill exists to remove.
