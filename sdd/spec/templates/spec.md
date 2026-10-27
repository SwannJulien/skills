# US-NN — <Title: the outcome, not the task>

[← Feature](../README.md)

| | |
|---|---|
| **Feature** | <feature name> |
| **Status** | draft \| approved — <YYYY-MM-DD, set only when the user approves> |
| **Kind** | feature \| enabler |
| **Depends on** | <US-NN, or — if independent> |
| **Blocks** | <US-NN, or —> |
| **Scope** | <layers this honestly touches, e.g. back + db> |
| **Breaks existing behavior?** | <No / Yes — what changes for current users> |

---

## Story

<!-- feature stories only; delete and use Objective for an enabler -->

**As a** <role — a real user of the system, not "a developer">
**I want** <capability>
**so that** <the consequence of not having it>

## Objective

<!-- enabler stories only; delete for a feature story -->

<One paragraph: what this makes possible, and why it has to happen before the
stories that depend on it.>

## Context

<What exists today. Quote the code or the schema — path, and the lines that
matter. If the current behavior is wrong, show it rather than describing it.>

```<lang>
// <path/to/file.ext>
<the code as it is now>
```

<A table is often the clearest way to show where current behavior breaks:>

| Layer | What happens today |
|---|---|
| | |

<Link the project rules that constrain this story.>

---

## Decisions and non-goals

<Choices made while specifying, and what was rejected. Later stages never see
the conversation this came out of — if a rejected alternative isn't written here,
it will be proposed again.>

- **<Decision>** — <why; and what was rejected>.
- **Assumed, not confirmed**: <assumption the reviewer should correct if wrong>.

**Out of scope for this story**: <what a reader might reasonably expect here and
will not find, and where it lives instead>.

---

## Scenarios

<Every externally observable behavior, one scenario per decision. Failure and
edge cases included. This is the only section that describes behavior.>

### Scenario: <the observable outcome, stated as fact>

```gherkin
Given <the precondition, concretely>
When <the single action>
Then <what is observable afterwards>
And <...>
```

### Scenario: <a failure case>

```gherkin
Given <...>
When <...>
Then <...>
```

---

## Acceptance criteria

<Everything that is NOT behavior. If a line would make sense as Given/When/Then,
it belongs in Scenarios instead. These checkboxes are ticked by the
implementation stage — they are the completion signal.>

- [ ] <contract shape: endpoint, status codes, payload fields>
- [ ] <error types and where they are raised>
- [ ] <where the code lives / which conventions it follows>
- [ ] <reference collection, fixtures or docs updated>
- [ ] <lint and tests pass>

---

## End-to-end verification

<One ordered walkthrough a human can actually perform, in the interface the
story's user uses. Replays the scenarios as a single sequence — it does not
restate them.>

**Setup**: <fixtures, accounts, state needed before step 1, loaded with the
project's own commands>

1. <action> → <what you should see>
2. <action> → <what you should see>
3. <action that shows the outcome lasted — reload, sign in again, open it
   elsewhere in the app> → <what you should see>

**Cleanup**: <how to undo the setup>

---

## Watch out for

<Traps specific to this story. The tempting wrong fix and why it is wrong. The
hazards the project's own documentation already records and that this story will
walk into. Not general advice.>

- **<The trap, stated as an imperative>** — <why it is a trap, concretely>.
