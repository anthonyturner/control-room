# Testing

Read this before you write or change a test. The engineer uses it to write
tests that mean something and to show that they do; the reviewer uses it to
check that proof. It is not a skill and not an opt-in workflow: it applies
whenever a change has logic to test.

The technique below is adapted from
[mattpocock/skills](https://github.com/mattpocock/skills) (MIT),
`skills/engineering/tdd`, restated in the playbook's words with a project
addendum appended; the licence notice is in
[third-party-notices.md](third-party-notices.md). This playbook differs from
upstream in the addendum, and in the technique by keeping refactoring on green
inside the loop.

## The technique

- **Test what a caller can see.** Name a test as a sentence about something a
  caller can do ("an expired card is declined at checkout"). Drive the code
  through the same entry point that caller uses, and check only what comes
  back or what the caller could notice. Rewriting the internals without
  changing behaviour should leave such a test green; if it turns red, the test
  knew too much. Likewise, never confirm an outcome by looking behind the
  interface (querying the database yourself, say): if the interface cannot show
  you the result, the interface has a gap.
- **Fake only what lies outside the process.** Stand in for things you do not
  control or cannot run cheaply: remote services, payment and mail providers,
  the clock, random numbers, and at times the filesystem or a database, though
  a real throwaway database often serves better. Your own modules are never
  faked; use them as they are. Hand a piece of code its outside dependency
  from the caller, as a parameter or constructor argument, instead of
  creating it inside, so a test can pass in a stand-in. Wrap each outside
  operation in its own small function, so each stand-in has to produce only
  one kind of result.
- **No test that cannot fail.** A test cannot fail when it computes its
  expected value with the same logic as the code under test, or compares a
  value with itself. Take expected values from outside the code: a literal you
  have checked by hand, a worked example, the specification.
- **One test, one slice, at a time.** Write a single test, do the least that
  makes it pass, and let what you learn shape the next test. Writing a whole
  batch of tests first is a trap: you would be guessing at behaviour not yet
  built, and locking in a design before it has been tried.
- **Red, then green.** Run the new test before the change and see it fail, for
  the reason you expected. Then write just enough code to turn it green. Code
  that no test asks for waits until a test does.
- **Refactor only on green.** With every test passing, tidy in small steps and
  re-run after each one. Do not change behaviour and structure in the same
  step.

Deep modules, and where a seam belongs, are in
[design-principles.md](design-principles.md). A test that has to reach past the
interface says the module has the wrong shape; fix the shape rather than the
test.

## Project addendum

**Seams are the engineer's call.** A seam (the place where a test calls in, or
where a fake may stand in) is chosen by the engineer, not confirmed with the
requester first. State the seams in one line in the technical approach and
again in the pull-request body ([rule 14](rules.md)). Stop only under the
conditions rule 14 already names.

**Proof of change.** A test is proof only if it fails without the change and
passes with it. For each new behaviour, record the test's name, the failure
without the change (the failing assertion, for example `expected 3, got 0`),
and that it passes with the change. There are three acceptable proofs. Proof 1
is the default. Where the project has CI, prefer to add proof 2 as well. Use
proof 3 only where proof 1 was not possible.

1. **Test-first red, captured. The default.** The first run before the change
   is the proof, at no extra cost. Write down the name and the failing line.
2. **Red CI on a pushed test-only commit.** Preferred in addition to proof 1
   where the project has CI, never required. The reviewer can check it without
   trusting you. Push the test alone, let CI go red, then push the change. It
   leaves a red commit in history, so skip it where a pre-push test hook would
   block the push.
3. **Revert-and-run. The fallback**, for when proof 1 was not possible, mainly
   the `open-pr` path, where code exists before any test. Back out the source
   change but not the test, run, restore the change, and run `git diff` to
   confirm nothing was lost. Touch only the issue's files ([rule 5](rules.md)).

The red has to be the behaviour assertion failing, not a compile error or a
missing import. Where the language allows, add a stub first so the test fails
on the assertion.

Not acceptable as proof: a test you never saw fail; a failure unrelated to the
change; or "this would fail", asserted without running it.

**Exceptions, each stated with its reason.**

- Styling, markup, and flows that can only be checked by hand in the running
  app.
- Documentation and configuration.
- A refactor that keeps behaviour: name the existing tests that cover it and
  that still pass.
- A defect fix: the regression test that reproduces the reported symptom is
  the proof.

**In the pull request.** One line under Verification per new behaviour, for
example "`checkout rejects an expired card`: failed without the change
(`expected 'declined', got 'confirmed'`), passes with it". With several new
behaviours, keep one such line for each, however brief: a reviewer cannot check
a test that is not named. For an exception: "exception: <reason>".
