# The test-writer's brief

The test-writer starts cold and sees none of the build conversation. Everything
below goes into its brief — the paths it must read, and these rules verbatim or
by absolute path to this file.

## What it gets

1. **The spec**, by path. Read it whole: the scenarios are what the tests
   assert, and **Decisions and non-goals** says what they must not.
2. **The plan's Test map section**, pasted into the brief. Not the plan's path:
   the steps are the implementer's route, and a test written from the route
   tests the route.
3. **The test levels** from `.claude/spec/conventions.md`, and the test
   commands and hazards from the project's agent instructions.
4. **Any failure or defect** being sent back, when this is a return trip: the
   test, the output, the scenario it was checked against.

## Rules

- **One test per row of the test map, at the level and path it names**, named
  `US-NN: <scenario title>` — verbatim, so the scenario can be found from the
  test and the test from the scenario. A scenario may need more than one test;
  each carries the same name, with a suffix. When the path is an existing file,
  add to it; don't open a sibling file for this story.
- **Assert the scenario's `Then` and `And` lines, and nothing the spec does not
  say.** A test asserting an implementation detail — a private function, a CSS
  class, the order of two unrelated calls — fails the day the code is improved,
  and teaches the implementer to distrust the suite.
- **Drive the entry point the test map names**, through its real interface. If
  it does not exist yet, code against it exactly as named; the test must then
  fail until the code lands. Do not stub what the test is meant to prove.
- **Read the existing tests first, and look like them** — they, not any
  document, are the source of truth for how tests are written here. Reuse their helpers,
  fixtures, page objects and setup. A second way of logging in a test user is a
  second thing to maintain.
- **Rewrite the tests listed under "Existing tests this story changes"** to
  assert the scenario that supersedes them. Do not delete one unless the spec
  removes the behavior it covers.
- **Write no production code**, and edit nothing outside the test directories
  and test fixtures. If a test cannot be written without a change to production
  code, say so and stop on that test.
- **Do not weaken a test to make it runnable.** If a scenario cannot be
  expressed as a test with this harness, report it; don't assert something
  nearby instead.
- **Run each new test once.** It is expected to fail. Report how it fails.

## What it returns

- The files written or changed.
- One line per test: its name, its scenario, and how it failed on its run.
- Each scenario it could not test, and why.
- Anything in the spec it found ambiguous — quoted, with the reading it chose.

Nothing else: no summary of the spec, no advice on the implementation.
