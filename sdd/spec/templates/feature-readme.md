# <Feature name>

<One paragraph: what this feature is, and what the system can do once every
story is delivered that it cannot do today.>

## Why now

<What makes this worth doing — the problem it closes, or what it unblocks.>

---

## Decisions that span the whole feature

<Choices that apply to several stories, recorded once here rather than repeated
in each. Individual stories link back to this section.>

- **<Decision>** — <why; what was rejected>.

**Out of scope for this feature**: <...>

---

## User stories

The order is the recommended implementation order. Each story is verifiable end
to end on its own and shippable without the next one; its **End-to-end
verification** section describes the exact walkthrough.

| # | Title | Kind | Scope | Depends on |
|---|---|---|---|---|
| [US-01](US-01-<slug>/US-01-<slug>-spec.md) | <title> | feature | <layers> | — |
| [US-02](US-02-<slug>/US-02-<slug>-spec.md) | <title> | enabler | <layers> | US-01 |

---

## Shared setup for verification

<Fixtures, accounts or state that several stories' verification walkthroughs
need, written once here. Include the cleanup — a test account that has been used
in a transaction is often no longer deletable, and that is worth knowing before
creating twelve of them.>

---

## Known hazards

<Things that will bite during this feature regardless of which story you are on:
project rules in play, places where the existing code is already wrong,
constraints from the data model.>
