# Personal AI Skills

In this repository you'll find the skills I have developed and I'm using in my personal projects

## Index

  1. [Installing a skill](#installing-a-skill)
  2. [/add-note](#add-note)
  3. [/line-by-line](#line-by-line)
  4. [/spec, /plan, /build](#spec-plan-build)

## Installing a skill

Every skill in this repository is a directory containing a `SKILL.md`, plus any
`templates/` and `references/` it reads while it runs. Installing one means
putting that directory — whole — where your agent looks for skills. The
directory name becomes the command, so `spec/` is invoked as `/spec`.

Keep the subdirectories with their `SKILL.md`. A `SKILL.md` copied on its own
will load and then fail to find the files it refers to.

### Globally, for every project

Agents read user-level skills from a directory under your home: `~/.claude/skills/`
for Claude Code, `~/.copilot/skills/` for Copilot, and the same pattern under
whichever dotfile directory your agent uses — check its documentation if you are
unsure. Create it if it doesn't exist, then either copy the skill in:

```bash
cp -r <skill> <agent-skills-dir>/
```

or symlink it, so that pulling this repository updates the installed skill:

```bash
ln -s "$(pwd)/<skill>" "<agent-skills-dir>/<skill>"
```

Symlink while you are still changing the skill — the edit is live on the next
invocation. Copy when you want a frozen version that your edits here won't
disturb.

### In a single project

Put the skill directory in the project's own skills folder — `.claude/skills/`,
`.copilot/skills/`, or your agent's equivalent — and commit it:

```bash
mkdir -p <project>/.claude/skills
cp -r <skill> <project>/.claude/skills/
```

The skill then travels with the repository and behaves identically for everyone
working on it, which is what you want once a team shares a methodology. Prefer
the global install when you are the only user, or when every project should pick
up changes at once.

Both can coexist: a project-level skill takes precedence over a global one with
the same name, so a project can pin its own version.


## /add-note

### Purpose

Turns something you just learned while chatting with an AI (a best practice, a gotcha, a rule of thumb, the difference between two approaches) into a lasting entry in a personal notes file, organized by topic (JS, Java, PostgreSQL, Docker, ...).

Entries are written for a beginner: they explain what the concept is, why it matters, how it works and when to use it, instead of just stating the rule. The result is a set of personal reference files that are easy to reread months later.

### Setup

Install `add-note/` as described in [Installing a skill](#installing-a-skill). The notes themselves are kept in a separate directory of your choice (typically their own repository), not in this repo.

### How it works

Invoke it with a topic and the knowledge to note, e.g. `/add-note JS the difference between == and ===`. If no topic is given, the skill asks for one.

1. **Find the learning directory.** On first use the skill asks where your notes are kept and saves the answer to `~/.config/add-note/config`. Later calls reuse it silently. You can target another directory for a single call by including its path in the request, or ask the skill to change the saved one.
2. **Find the topic file.** The topic maps to `{topic-lowercase}/{topic-lowercase}_notes.md` inside the notes directory (e.g. `javascript/js_notes.md`). If the file doesn't exist, it is created from a standard skeleton with an Index and an Entries section.
3. **Read the whole file** to know the current entry count, existing titles, and whether the notes are flat or already split into chapters.
4. **Check for duplicates.** If a similar entry already exists, the skill asks whether to expand it or add a new one.
5. **Generalize the content.** Project-specific names, business logic, internal URLs and other proprietary details are stripped out so the entry reads as generic guidance.
6. **Write the entry**, numbered sequentially, with:
   - a teacher-like explanation (short paragraphs, bullets, tables or Do/Don't pairs where useful)
   - a small, well-commented code example when it adds clarity
   - a selective line-by-line walkthrough for code a beginner might not follow
   - a Mermaid, table or ASCII diagram for flows, lifecycles or comparisons
   - a **Further reading** link to an authoritative source
7. **File it** at the bottom of the flat list (and the Index), or under the matching chapter if the notes have been reorganized.
8. **Offer to reorganize.** Once the notes reach about 15 entries, or 3+ entries cluster around the same subject, the skill proposes grouping them into chapters, and only does it after you confirm.
9. **Confirm** which entry was added, the full path of the file, and where in it.


## /line-by-line

### Purpose
### How it works


## /spec, /plan, /build

### Purpose

Spec-driven development in three stages, with a human gate between each. One
skill writes the specification, a second turns one story into an implementation
plan, a third builds it. Project-agnostic.

| Stage | Skill | Writes | Reads |
|---|---|---|---|
| 1 | `spec` | `README.md` + `US-NN-*.md` | the repo + you |
| 2 | `plan` | `US-NN-*-plan.md` | one spec + the repo |
| 3 | `build` | code | one spec + its plan |

Each stage starts with clean context and hands off through a document. Nothing
travels between stages except what is written down — which is why the spec is
required to record its own decisions and the alternatives it rejected.

### Setup

Install all three as described in [Installing a skill](#installing-a-skill);
they are meant to be used together. The first run in a new project writes
`.spec/conventions.md` at its root — where specs live, how the project is built,
tested and migrated, what must not be run without asking. Commit it: all three
skills read it, and it is what keeps project plumbing out of the interview.

### How it works

```
/spec "add multi-library support"
    → reads the code, asks up to 10 questions
    → proposes a cut into stories, and stops
  you: approve or re-cut
    → writes one document per story

  you: read the specs, correct, validate            ← the gate that matters

/plan US-01                 (clean context)
    → writes US-01-…-plan.md
  you: skim

/build US-01                (clean context)
    → builds it, ticks the acceptance criteria, runs the walkthrough
```

Repeat the last two per story. Re-run `/spec` only when the feature changes
shape.

Each story document carries the same sections: the user story or objective, the
context quoted from the code as it is today, the decisions and non-goals,
Gherkin scenarios for every observable behavior, acceptance criteria as
checkboxes, an end-to-end verification walkthrough, and the traps to watch for.

### Design decisions

**A story is "independently verifiable and shippable", not "touches every
layer."** A layer quota makes the agent pad backend stories with UI work to
qualify. The story records the layers it honestly touches.

**Two kinds of story.** `feature` gets a user story; `enabler` (migrations,
infrastructure) gets a plain objective, because forcing a user story onto one
yields "As a developer, I want Flyway". Enablers are budgeted at roughly one in
four — more than that means the cut is wrong.

**The cut is approved before the documents are written.** Re-cutting a proposal
is free; re-cutting twelve finished documents is not, and a human asked to review
twelve documents at once reviews none of them.

**One job per section.** Scenarios hold all the behavior. Acceptance criteria
hold everything that isn't behavior. The end-to-end walkthrough replays the
scenarios rather than restating them. When a fact appears twice, one copy goes
stale and the reader can't tell which is normative.

**The plan is a separate, disposable document.** It cites line numbers and goes
stale on first edit; the spec stays true after merge. Keeping them in one file
also turns the behavior review into a code review. Durable knowledge found while
planning gets flagged for promotion into the spec's "Watch out for".

**Checkboxes are the handoff protocol.** `- [ ]` → `- [x]` in the spec is the
only edit the implementer may make to it, and the only mechanical completion
signal in the pipeline.

**The spec is written once and amended only by a human.** If implementation shows
the spec is wrong, the stage stops and reports. A spec edited to match the code
is a description, not a specification.

**The spec agent may argue back.** If the request is really one story, or three
features under one name, or already built, it says so with the evidence — once.
If you reaffirm, the disagreement is recorded under "Decisions and non-goals" so
later stages don't reopen it.

