# Bootstrapping `.claude/spec/conventions.md`

Runs once per project, the first time any of the three skills is used. Its
purpose is to keep project plumbing out of the interview: nothing recorded here
should ever cost one of the ten questions again.

Two files carry what the three stages need, and nothing is recorded in both:

| File | Holds | Owner |
|---|---|---|
| The project's agent instructions (`CLAUDE.md` or `.claude/CLAUDE.md`, `AGENTS.md`) | languages, build/test/lint commands, how the app runs, schema changes, commands that need permission, hazards | the user — every agent and every task reads it |
| `.claude/spec/conventions.md` | where specs live, the layers a story can name, which test level proves what, how a walkthrough is performed, criteria every story carries | this skill — only spec, plan and build read it |

The test for where a fact goes: **would an agent fixing a bug with no spec need
it?** If yes, it belongs in the agent instructions.

## Step 1 — Check the agent instructions

Read the project's agent instructions and check they cover what the stages rely
on:

| Fact | Where to look if it is missing |
|---|---|
| Doc language and code language | existing docs under `docs/`, `README.md`, identifiers and commit messages |
| Test commands — one per level the project has (unit, integration, end-to-end), and how to run a single test | build and package manifests, task runners, `Makefile`, CI workflow files |
| Lint command | same places |
| How the app runs | `docker-compose.yml`, `README.md`, run configurations |
| Database access and migrations | `docker-compose.yml`, container names, migration tool and directory |
| Commands that need permission | formatters that rewrite untouched files, browser installs, anything destructive |

For each missing fact, detect what you can from the repo. Then **propose** an
addition to the agent instructions, using the matching sections of
`templates/claude-md.md`, and apply it only with approval — it is their
instruction file, not yours. Ask about what you could not detect, above all
**how the app is run**, since that shapes every end-to-end walkthrough.

If there are no agent instructions at all, offer to create `CLAUDE.md` from
`templates/claude-md.md`. If the user declines either offer, carry on: later
stages detect commands from the repo themselves and say what they guessed.

## Step 2 — Detect the spec conventions

| Field | Where to look |
|---|---|
| Specs root | an existing specs/stories/docs tree; otherwise propose `docs/specs/` |
| Layers | `package.json`, `pom.xml`, `go.mod`, `Cargo.toml`, directory layout at the repo root |
| Walkthrough interfaces | a UI, `*.bru`, `*.http`, `*.rest`, Postman/Insomnia exports, a database shell |
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

## Layers

<The layers a story's Scope field can name, e.g. `db`, `back`, `front`,
with the directory each lives in.>

## Tests

Every scenario in a spec becomes at least one automated test. Commands are in
the agent instructions; this section says which level proves what.

| Level | Proves | Lives in | Harness |
|---|---|---|---|
| unit | <logic in isolation> | `<dir, naming pattern>` | <tool, or `missing`> |
| integration | <a layer through its real interface, e.g. the API against a real database> | `<dir>` | <tool, or `missing`> |
| end-to-end | <the system from outside, as a user drives it> | `<dir>` | <tool, or `missing`> |

- **End-to-end setup**: <what must be running, and the data the tests start
  from; how to start it is in the agent instructions>
- **Naming**: every scenario test carries `US-NN: <scenario title>` in its name
  or description, so a spec scenario can be found by searching the tests

## End-to-end walkthrough

- **Interfaces**: <what a walkthrough goes through, in order of preference —
  the UI, the HTTP collection at `<path>`, the database shell>
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
