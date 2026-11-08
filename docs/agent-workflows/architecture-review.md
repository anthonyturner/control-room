# Architecture review workflow

An architecture review looks across the whole codebase, not at one change, for
shallow modules ([design-principles.md](../design-principles.md)): places where
learning the interface costs about as much as the work it hides. It ranks where
deepening would pay most, lets the requester pick one, stress-tests the shape
of that change with them, and ends with a written request for
[planning.md](planning.md). It changes no code.

**It is opt-in.** It is the `improve-codebase-architecture` skill, and it runs
only when the requester asks for it by name. It is not a gate, not part of any
other stage, and never something an agent starts on its own initiative — see
[rule 13](../rules.md), which counts this skill as a by-name way into the
grilling in step 4.

The name is the upstream skill's, kept under the exception in
[skills.md](skills.md). It promises more than the review does: the review only
proposes, and the improvement is a later `refactor` issue.

The technique is adapted from
[mattpocock/skills](https://github.com/mattpocock/skills) (MIT),
`skills/engineering/improve-codebase-architecture`, with the shared vocabulary
of its `codebase-design` skill, restated here in the playbook's words with a
project addendum appended; the licence notice is in
[third-party-notices.md](../third-party-notices.md).
Where this playbook differs from upstream is in the addendum.

## Vocabulary

The design words are defined in [../design-principles.md](../design-principles.md):
module, interface, depth, seam, leverage, locality and the deletion test, plus
the four kinds of dependency. Use them as defined whenever you describe the
shape of a design. Name the domain in the project's own words: if
`docs/glossary.md` exists, write "the invoice numbering module", not "the
InvoiceService". Names that belong to the project's stack stay as they are.

"Adapter" is not a design term in this playbook. It already means a per-tool
wrapper ([skills.md](skills.md)), so describe what sits at a seam as an
implementation.

## Process

### 1. Choose where to look

Look where change is happening, because deepening pays off through future
changes. If the requester named an area, a module or a pain point, take that and
skip the rest of this step. Otherwise let history point the way. Read a bounded
slice of it, for example
`git log --max-count=300 --since="6 months ago" --name-only --pretty=format:`,
and count which paths recur, by piping through `sort | uniq -c | sort -rn` or by
tallying them yourself. The paths that recur most get attention first. If no
path stands out, widen the slice once, by date or by count. If still nothing
stands out, or the repository is young, or the tool cannot run `git`, ask the
requester to name an area. Do not scan the whole tree unasked; it costs far more
than the question.

Before looking at code, read the area's decision records in
[../decisions/](../decisions/README.md) and `docs/glossary.md` when it exists.
A decision record is settled; do not re-open one without friction strong enough
to warrant it.

### 2. Explore

Walk the chosen area and note where it hurts. Where the tool has subagents,
hand the walk to a read-only exploration subagent, briefed with the area, the
vocabulary above and the glossary terms; it returns findings and edits nothing.
Where it does not, do the walk in the conversation. Do not follow a rigid
checklist, but these questions find most of it:

- Where does following one idea mean opening a string of tiny files, such as a
  validator, a mapper and a helper that each do a sliver?
- Which modules are shallow, with an interface almost as hard to learn as the
  code behind it?
- Where was logic pulled out only so a test could reach it, while the real bugs
  live in how it gets wired up? Those tests pass and the failures stay.
- Which modules know each other's internals so well that a change to one forces
  an edit to the other?
- What is untested, or testable only by reaching around the interface?

Apply the deletion test to anything that looks shallow. A module whose removal
would scatter its work over its callers is earning its keep. One whose removal
leaves its work in a single sensible place, or leaves nothing, is a candidate.

### 3. Present the candidates

Show them in the reply, ranked, at most five. Write nothing to disk. For each:

- **Name**, in domain words, and the **files** involved.
- **Problem**: what hurts now, in a sentence.
- **Change**: what would be different, in plain words. Propose no interface yet.
- **Gain**: who benefits and how, named as locality (what stops being spread
  over many callers), leverage (what callers no longer need to learn) and
  testing (which tests become possible, and which go away). "Easier to
  maintain" does not count.
- **Strength**: `Strong`, `Worth exploring` or `Speculative`.
- **A before and after diagram**, drawn as [../diagrams.md](../diagrams.md)
  describes: ASCII in a terminal chat, a Mermaid block where it renders. One
  diagram per candidate, even when a candidate would be a large change. The
  HTML page in that file is not used here, because the review writes no file;
  the issue that planning files carries the candidate's diagram.
- **Conflict with a decision record**, only when the candidate contradicts one.
  Name the record and say why the friction is worth re-opening it. A record
  that merely rules out some other refactor does not belong in the list.

Finish with the one you would do first and why, then ask which the requester
wants to explore.

### 4. Grill the pick

Once the requester picks one, run a grilling session on that candidate alone,
in the conversation, following [grilling.md](grilling.md). The questions to
settle are the constraints, what the module depends on, what its interface
would be, what hides behind it, and which of today's tests still make sense.

- **Sort the dependencies** into the four kinds under "Dependencies across a
  seam" in [../design-principles.md](../design-principles.md). The kind decides
  where the seam goes and how tests stand in for what is behind it. Add no seam
  that would have a single occupant.
- **Replace tests, do not layer them.** Tests of the shallow modules that the
  new interface makes redundant are deleted, and new tests cross the new
  interface and assert on what a caller can observe.
- **Alternative shapes.** If the requester wants them, sketch two or three
  radically different interfaces and compare them by depth, locality and where
  the seam falls, then give a recommendation. A subagent may draft shapes
  where the tool has one; the comparison and the choice stay in the conversation.
- **A lasting rejection.** If the requester turns a candidate down for a reason
  a later review would need to avoid suggesting it again, offer to record it as
  a decision record. Skip reasons that are passing ("not now") or obvious. The
  review never writes the record; if the requester accepts, it is written
  through an issue and pull request as [../decisions/README.md](../decisions/README.md)
  describes.
- **New words.** A concept the glossary lacks goes into the request as a term to
  add; the review edits no glossary. The requester can run a
  `grill-with-docs` session ([domain-modeling.md](domain-modeling.md)) to settle
  language first.

### 5. Hand off

When the grilling settles, write the request for `plan-issue` in the reply, and
stop. It holds:

- the **problem**, with its evidence: the files, and what the history showed;
- the **proposed shape**, and the diagram from step 3;
- the **settled decisions** from step 4, including those that ruled things out;
- what is **out of scope**;
- **candidate acceptance criteria**, including that behaviour is unchanged and
  that tests cross the new interface.

Say that the work is a refactor, so its branch type is `refactor`
([tracking.md](tracking.md)). Invent no label; planning uses the repository's
real ones. Close with a handoff line naming `/agent-playbook:plan-issue`.
In Codex it is `$plan-issue`.
In Copilot it is the `orchestrator` custom agent, asked to plan only
([skills.md](skills.md)).
Do not run planning as part of the review; that is the requester's call.

## Project addendum

**When it runs.** Only when the requester asks by name. A requester who runs the
skill has asked for the grilling in step 4, and for nothing more: no other
grilling starts from it. Like grilling, it is never a gate, and no stage may
refuse to proceed for want of a review.

**Where it runs.** In the main conversation with the requester. The exploration
in step 2 may go to a read-only subagent where the tool has one. The grilling
never does, because a subagent has no channel to the requester.

**What it writes.** Nothing. No repository file, no temporary file, no HTML
page, no browser, no branch, no commit, no issue and no comment. The only
commands it runs are read-only `git` queries over history and file listings. Its
whole output is the reply, so a review that ends without a handoff has lost
nothing.

**Where it differs from upstream.** Upstream writes a styled HTML report to a
temporary directory and opens it; this playbook draws each candidate inline and
writes no file. Upstream updates the glossary and offers decision records as it
goes; here the glossary is left alone, and a decision record is offered, never
written. Upstream gets its vocabulary from a separate design skill; here it is
[../design-principles.md](../design-principles.md), so there is one definition.
Upstream stops after the grilling; here the review ends in a request for
`plan-issue`.

**Relation to `/smell`.** The plugin's `/agent-playbook:smell` gives line-level
findings on the branch diff, before merge. This review gives module-level
candidates from the history's hot spots, and one of them becomes the next
refactor's request. They overlap only where a smell finding names a module that
is shallow.

**Defects.** A review is not how a defect is diagnosed; that is engineering's
job under **Defect lane** in [pipeline.md](pipeline.md).

**Question format.** Follow [../response-style.md](../response-style.md) in the
candidates, the interview and the handoff: plain English first, technical detail
underneath, and a repository-specific name explained the first time it appears.
