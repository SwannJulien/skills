# US-NN — Implementation plan

[← Spec](US-NN-<slug>-spec.md) · [← Feature](../README.md)

> Disposable. The spec is the durable document; this plan is a route through the
> code as it stood on <date> and may be deleted once the story ships.

## State of play

<What already exists of this story, from reading the code. What is missing, what
is half-built, what is built and wrong. This is the section that saves the
implementer from rediscovering it.>

---

## Design choices

<Only choices a reasonable engineer could have made the other way that are hard
to reverse, add a dependency, change a contract outside this story, or depart
from the project's pattern. "None" is a common answer.>

- **<The choice>** — rejected: <the alternative>, because <why>. Reversing it
  costs: <a migration, a contract change, …>. Later stories: <US-NN it serves or
  constrains, and how; or none>.
  ADR draft, if it is hard to reverse, surprising, and a real trade-off:
  > <context, decision, why — one to three sentences>

---

## Test map

<Self-contained: build hands this section alone to the test-writer, who never
reads the steps. One row per spec scenario.>

| Scenario | Level | Test file | Drives | Setup |
|---|---|---|---|---|
| <scenario title, verbatim> | unit \| integration \| end-to-end | `<path>` | <endpoint + method, route + elements, or function signature> | <fixtures, accounts, existing helpers> |

### Existing tests this story changes

| Test | Asserts today | Superseded by |
|---|---|---|
| `<path>` — <test name> | <old behavior> | <scenario title> |

### Not automated

<Scenarios no test can observe with this project's harness, and why. "None" is
the expected answer.>

---

## Steps

<Ordered. Each step leaves the project in a state that compiles.>

### 1. <What this step achieves>

<Where, and what changes. Real paths and identifiers.>

```<lang>
// <path/to/file.ext>
<only the signature, query or config line that prose would leave ambiguous>
```

<Anything non-obvious about this step — the ordering constraint, the trap, the
reason the obvious version does not work.>

### 2. <...>

---

## Acceptance criteria trace

<Every criterion from the spec, mapped to the step that satisfies it. A criterion
with no step is a hole; a step serving no criterion is scope creep or a missing
criterion — say which.>

| Spec criterion | Step |
|---|---|
| <criterion, abbreviated> | 1, 3 |

---

## Where this is likely to break

<The steps you expect to fail first, and what the failure will look like. Being
able to recognise the expected failure is worth as much as avoiding it.>

---

## To promote into the spec

<Durable knowledge found while planning — traps that will still be traps after
this story ships. The human moves these into the spec's "Watch out for" section;
this plan is deleted and would take them with it.>

- **<The trap>** — <why>.

## Problems found in the spec

<Anything impossible, self-contradictory, or resting on a false premise about the
code. Not fixed here — reported for the human to amend. "None" is a fine
answer.>
