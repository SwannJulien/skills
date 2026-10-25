<!--
  Template for a project's CLAUDE.md — the instructions every agent session
  loads before doing anything in this repository.

  Where it lives: `CLAUDE.md` at the repo root, or `.claude/CLAUDE.md`. Both are
  loaded. Commit it: it is shared by everyone working on the project. Personal
  preferences go in `CLAUDE.local.md` (gitignored) instead.

  What belongs here — the one test: **would an agent get this wrong without
  being told?** Things it can work out by reading the code don't belong here.
  Things it would guess wrong, or would have to find by trial and error, do.

  Rules for writing it:
  - Keep it short. It is loaded in every session, for every task, so every
    line costs context forever. Aim for under ~150 lines; link to longer docs
    rather than copying them (`see docs/architecture.md`, or `@docs/file.md` to
    import one).
  - Be specific. "Run `pnpm test -- path/to/file`" beats "run the tests".
    "Never edit an applied migration" beats "be careful with migrations".
  - Give the reason for any rule that looks arbitrary. An agent that knows
    *why* applies the rule correctly to cases you didn't list.
  - Don't restate what a linter or formatter already enforces, and don't write
    generic advice ("write clean code", "add tests"). Agents already do that.
  - Never put secrets here.
  - Update it when an agent gets something wrong twice: that is the signal
    a rule is missing.

  Delete every section that has nothing project-specific to say, and delete
  these comments once you have filled it in.
-->

# <Project name>

<!-- Two or three lines: what this is, who uses it, and the one thing that
     makes it unusual. Context for every decision the agent makes. -->

<What the project does, for whom.>

## Language

<!-- State both, even if they are the same — agents default to English. -->

- **Docs** (specs, READMEs, docs/): <language>
- **Code** (identifiers, comments, commit messages): <language>

## Layout

<!-- Only what isn't obvious from the directory names. Skip it entirely for a
     conventional single-app repo. -->

- `<dir>/` — <what lives there, if the name doesn't say>

## Commands

<!-- Exact, copy-pasteable. One line per test level the project has, and how
     to run a *single* test: agents run tests constantly, and the full suite is
     often too slow. Say what the end-to-end tests need running first. If a
     level has no tests, say so — that is information. -->

- **Install**: `<command>`
- **Run**: `<command>`
- **Test**: `<command>` — one file: `<command>`
- **Integration tests**: `<command>`
- **End-to-end tests**: `<command>` — needs <what must be running>
- **Lint**: `<command>`
- **Build**: `<command>`

## Running the app

<!-- The quirks. "The backend is started from the IDE, not the shell — don't
     start it, ask." "The front end expects the API on port 8080." What must be
     running before what. -->

- <quirk, and why>

## Database

<!-- How to open a shell, how to load or reset development data, the
     migration tool and its rules. -->

- **Shell**: `<command>`
- **Fixtures**: `<how to load or reset development data>`
- **Migrations**: <tool, directory, naming> — <what is forbidden, e.g. never
  edit an applied migration: it has already run in other environments>

## Code conventions

<!-- Only the house rules a linter doesn't catch and the code doesn't make
     obvious: error handling pattern, where business logic goes, which of two
     coexisting styles is the current one. -->

- <rule, and why>

## Git

<!-- Commit message format, branch naming, whether to commit at all without
     being asked. -->

- <rule>

## Ask before running

<!-- Commands that are slow, destructive, outward-facing, or that rewrite
     files nobody asked to change. -->

- `<command>` — <why it needs a human>

## Watch out for

<!-- The traps: the tempting wrong fix, the thing that looks dead but isn't,
     the behaviour that surprised the last person. One line each, with a path. -->

- <trap> — `<path>`

## Further reading

<!-- Pointers, not copies. -->

- `docs/map/` — generated map of each layer (`/map`)
- `.claude/spec/conventions.md` — where specs live (`/spec`, `/plan`, `/build`)
- <ADRs, architecture docs>
