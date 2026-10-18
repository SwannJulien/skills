# Bootstrapping `.spec/conventions.md`

Runs once per project, the first time any of the three skills is used. Its
purpose is to keep project plumbing out of the interview: nothing recorded here
should ever cost one of the ten questions again.

## Detect first, ask last

Work out as much as you can from the repo before asking anything.

| Field | Where to look |
|---|---|
| Doc language | existing docs under `docs/`, `README.md`; agent instructions often state it explicitly |
| Specs root | an existing specs/stories/docs tree; otherwise propose `docs/specs/` |
| Layers | `package.json`, `pom.xml`, `go.mod`, `Cargo.toml`, directory layout at the repo root |
| Test command | `package.json` scripts, `Makefile`, CI workflow files; **note if there is no test suite at all** |
| Lint command | same places |
| HTTP collection | `*.bru`, `*.http`, `*.rest`, Postman/Insomnia exports |
| Database access | `docker-compose.yml`, container names, migration tool and directory |
| How the app runs | agent instructions first — projects often have a rule like "the backend starts from the IDE, not the shell" |
| Hazards | `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md`, ADRs |

Then ask the user only about what you could not determine, and about anything
where guessing wrong would waste their time — above all **how a change gets
verified in this project**, since that shapes every end-to-end verification
section you will ever write here.

If the project has no automated tests, record that plainly. It is not a gap to
apologise for; it is the fact that makes the end-to-end verification walkthrough
the only real check, and the spec must be written accordingly.

## File to write

```markdown
# Spec conventions

Used by the `spec`, `plan` and `build` skills. Edit freely — these
are read, not enforced.

## Documents

- **Language**: <language the spec documents are written in>
- **Specs root**: `<path>` — one folder per feature `FEAT-<slug>/`, holding
  `README.md` and one folder per story `US-NN-<slug>/` with
  `US-NN-<slug>-spec.md` and `US-NN-<slug>-plan.md` inside
- **Numbering**: per feature folder, from `US-01`
- **Code language**: <language of identifiers, comments and commit messages>

## Layers

<The layers a story's Scope field can name, e.g. `db`, `back`, `front`,
with the directory each lives in.>

## How a change is verified

- **Automated tests**: <command, or "none — this project has no test suite">
- **Lint**: <command>
- **HTTP collection**: <tool and path, and whether a new endpoint requires a new file>
- **Database**: <command to open a shell>
- **Running the app**: <how the services actually start, including anything that
  must not be done from the shell>
- **Fixtures**: <how to load or reset development data>

## Schema changes

<Migration tool, directory, naming, and what is forbidden — e.g. never editing
an applied migration.>

## Hazards to carry into every spec

<Pointers to the project's own rules, with paths. The spec skill reads these;
list them rather than copying them, so they stay in one place.>

## Tooling that needs permission

<Commands that must not be run without asking — formatters that rewrite
untouched files, browser installs, anything destructive.>
```

## After writing

Show the user a summary — specs root, verification commands, anything you had to
guess — and get confirmation before continuing to the interview. A wrong specs
root discovered after twelve documents are written is an annoying thing to fix.
