# US-NN — Review

[← Spec](US-NN-<slug>-spec.md) · [← Plan](US-NN-<slug>-plan.md) · [← Feature](../README.md)

| | |
|---|---|
| **Round** | <1, 2, …> — <YYYY-MM-DD> |
| **Diff reviewed** | <working tree against `<commit>` \| `<base>..<head>`> |
| **Files outside the story** | <none, or the paths, and whether they were scope creep or someone else's work> |
| **Verdict** | ready for walkthrough \| back to build — <N> blocking |

> Review writes the findings; build fills **Status** when it answers them, and
> the next round checks them. Nothing else in this document is edited by build.

---

## Criteria unticked

<Each criterion build ticked that does not hold, with the evidence. "None" is
the expected answer.>

- **<criterion, abbreviated>** — <what was run, what it showed>.

## Lint and tests

<The commands run, per level, and the result. Failures with their output.>

| Level | Command | Result |
|---|---|---|
| lint | `<command>` | pass \| <N> errors |
| <unit \| integration \| end-to-end> | `<command>` | <passed / total> |

---

## Findings

<Blocking first. File and line, what is wrong, the evidence, and the spec line,
plan entry or rule it is judged against. One sentence of fix at most.>

### Blocking

| # | Where | Finding | Judged against | Status |
|---|---|---|---|---|
| B1 | `<path>:<line>` | <what is wrong, and the evidence> | <scenario, criterion, decision, rule> | open \| fixed \| disputed — <build's reason> |

### Should fix

| # | Where | Finding | Judged against | Status |
|---|---|---|---|---|
| S1 | | | | |

### Notes

<For the user: documents that disagree with the code, problems outside the
story. Not for build.>

- **N1** — <…>

### Spec problems

<The code does what the spec says, and the spec looks wrong. Possible
amendments for the user — never fixed by review or build on their own.>

- **P1** — <the scenario or criterion, quoted; why it looks wrong>.

---

## Previous round

<Second round onwards: each earlier finding, resolved (with evidence), still
open, or disputed — and, for a dispute review still holds, that it goes to the
user.>

| # | Status now | Evidence |
|---|---|---|
| | | |
