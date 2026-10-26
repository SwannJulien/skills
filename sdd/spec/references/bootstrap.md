# Bootstrapping `.claude/spec/conventions.md`

Runs once per project, the first time any of the pipeline's skills is used. Its
purpose is to keep project plumbing out of the interview: nothing recorded here
should ever cost an interview question again.

Two files carry what the four stages need, and nothing is recorded in both:

| File | Holds | Owner |
|---|---|---|
| The project's agent instructions (`CLAUDE.md` or `.claude/CLAUDE.md`, `AGENTS.md`) | the facts in Step 1, in whatever shape the project already uses | the user — every agent and every task reads it |
| `.claude/spec/conventions.md` | where specs live, the layers a story can name, which test level proves what, how a walkthrough is performed, criteria every story carries | this skill — only spec, plan, build and review read it |

The test for where a fact goes: **would an agent fixing a bug with no spec need
it?** If yes, it belongs in the agent instructions.

## Step 1 — Check the agent instructions

Read the project's agent instructions and check they state the facts the
stages rely on. This is a list of facts, not of sections: the project's file
can hold them however it likes, and a fact that does not apply to this project
— there is no lint, nothing has state — is simply absent, not a gap.

| Fact | Why the stages need it |
|---|---|
| The language documents are written in, if not English | spec and plan write documents; agents default to English |
| How to build, lint and test — per test level, and how to run a single test | build verifies by running these, not by reading its diff |
| How to run the system, and what must be running first | end-to-end tests and the walkthrough drive the running system |
| How to put the system's state back to a known starting point, if it has any | every end-to-end test and walkthrough starts from it |
| How that state's shape is changed safely, if the project has rules about it | stories that change it must follow them |
| What must not be run without asking | build runs commands unattended |

For each missing fact, detect what you can from the repo — manifests, task
runners, CI workflows, README, run configurations — and ask about the rest,
above all **how the system is run**, since that shapes every end-to-end test.
Then **propose** the addition, written in the style of the existing file, and
apply it only with approval: it is their instruction file, not yours.

If there are no agent instructions at all, offer to create a minimal file
holding only these facts. If the user declines either offer, carry on: later
stages detect commands from the repo themselves and say what they guessed.

## Step 2 — Detect the spec conventions

| Field | Where to look |
|---|---|
| Specs root | an existing specs/stories/docs tree; otherwise propose `docs/specs/` |
| Glossary | an existing `GLOSSARY.md`, ubiquitous-language or domain-terms document; otherwise propose `GLOSSARY.md` at the root, created when the first term resolves |
| Layers | the project's manifests and the directory layout at the repo root |
| Walkthrough interfaces | whatever a person uses to drive and inspect the system — a UI, a request collection, a CLI, a shell onto its storage |
| Walkthrough setup | fixture files, seed scripts, test accounts named in docs |
| Test levels | existing test directories and their naming, test-runner configuration, browser-automation configuration, which levels CI runs |
| End-to-end harness | what drives the app from outside — a browser-automation tool for a UI, an HTTP client for an API, the binary itself for a CLI — and what must be running first |

Ask only about what you could not determine.

Every scenario becomes an automated test, so a missing test level is a gap to
record, not to work around. Mark each level the project lacks as `missing`. The
spec skill opens the next feature with an enabler story that sets it up. Do not
pick the tool silently: propose the conventional choice for this project's stack
— whatever its ecosystem treats as the default — and let the user confirm it.
These skills prescribe no stack.

## File to write

```markdown
# Spec conventions

Used by the `spec`, `plan` and `build` skills. Edit freely — these are read,
not enforced. Project-wide facts — languages, commands, how the app runs, what
must not be run — live in the project's agent instructions, not here.

## Documents

- **Specs root**: `<path>` — one folder per feature `FEAT-<slug>/`, holding
  `README.md` and one folder per story `US-NN-<slug>/` with
  `US-NN-<slug>-spec.md` and `US-NN-<slug>-plan.md` inside
- **Numbering**: per feature folder, from `US-01`
- **Status**: every story is `draft` until the user approves it at gate 1;
  plan refuses a story that is not `approved`
- **Glossary**: `<path>` — the project's vocabulary. The spec interview writes
  terms as they resolve; every stage uses them

## Layers

<The layers a story's Scope field can name, e.g. `db`, `back`, `front`,
with the directory each lives in.>

## Tests

Every scenario in a spec becomes at least one automated test. Commands are in
the agent instructions; this section says which level proves what.

<One row per level this project has or needs, from the smallest to
end-to-end. The names are the project's own.>

| Level | Proves | Lives in | Harness |
|---|---|---|---|
| <name> | <what a test at this level can observe> | `<dir, naming pattern>` | <tool, or `missing`> |
| end-to-end | <the system from outside, as its user drives it> | `<dir>` | <tool, or `missing`> |

- **End-to-end setup**: <what must be running, and the data the tests start
  from; how to start it is in the agent instructions>
- **Naming**: every scenario test carries `US-NN: <scenario title>` in its name
  or description, so a spec scenario can be found by searching the tests

## End-to-end walkthrough

- **Interfaces**: <what a walkthrough goes through, in order of preference —
  e.g. the UI, a request collection at `<path>`, a shell onto its storage>
- **Setup**: <the accounts and data a walkthrough starts from; how to load
  them is in the agent instructions>

## Standing acceptance criteria

- [ ] Every scenario has an automated test named `US-NN: <scenario title>`
- [ ] The whole test suite passes, end-to-end included
- [ ] Lint passes

<Others this project adds, e.g. "a new endpoint has a file in the HTTP
collection".>
```

## After writing

Show the user a summary — specs root, test levels and any marked `missing`,
walkthrough interfaces, any addition you
proposed to the agent instructions, anything you had to guess — and get
confirmation before continuing to the interview. A wrong specs root discovered
after twelve documents are written is an annoying thing to fix.
