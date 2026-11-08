# Tracking workflow (GitHub)

control-room tracks work in **GitHub only**. A GitHub issue is the work
item, its pull request is the change, and the two together are the complete
record. There is no second tracker, and no stage writes to one.

See [ADR-0002](../decisions/0002-track-work-in-github-only.md) for why,
including what this costs.

## Identity

Each work item is one GitHub issue, referred to by its number.

- The **branch** is `<type>/<issue-number>-<short-description>`, e.g.
  `feat/123-saved-filters`. `<type>` is the Conventional Commit type of the
  change, and nothing else: `feat`, `fix`, `docs`, `chore`, `refactor`,
  `perf`, `test`, `ci`.
- The **pull request body** contains `Closes #<number>`, so the merge closes the
  issue.
- Do **not** put a bare `#<number>` in a commit body — see **Issue references
  in commits** in [implementation.md](implementation.md). Reference it in
  prose ("issue 123") instead, and keep the link in the pull-request body
  where it belongs.

## Status

Status is the issue's open/closed state plus its pull request's state. There is
no separate status field to keep in step, and nothing to transition.

| Stage | After the stage |
| --- | --- |
| Product Manager files the item | Issue open, labelled |
| Engineering Refinement | Assessment posted as an issue comment |
| UX Design (optional) | Handoff posted as an issue comment |
| Software Engineer starts | Branch pushed |
| Draft pull request opened | Draft PR linked to the issue |
| Implementation verified | PR marked ready for review |
| QA review | Review posted as a PR comment; findings fixed |
| Merge (merge commit) | Issue closed by `Closes #n` |

**No agent closes an issue by hand.** Completion follows a merge. An agent that
closes an issue is asserting a result no merge has recorded. A wayfinding map or
ticket has no pull request, so a person closes it — see
[Wayfinding](#wayfinding-maps-and-decision-tickets).

## Labels

List the repository's real labels with `gh label list` before selecting any.
Never invent one. The set below is the vocabulary this workflow expects; a
repository that lacks one of these labels simply does not use it until a person
creates it.

| Kind | Labels |
| --- | --- |
| Type | `enhancement`, `bug`, `chore`, `documentation` |
| Priority | `P0`, `P1`, `P2`, `P3` |
| Area | `area:<name>`, one per part of the product, defined by the project |
| State | `needs-triage`, `needs-refinement`, `ready`, `blocked`, `duplicate` |
| Wayfinding | `wayfinding` |

- `needs-refinement` — scope is not agreed yet; not ready to start.
- `ready` — refined and ready to start.
- `blocked` — say what it is blocked by in the issue body, and link that issue.
- `duplicate` — comment with a link to the original. A person closes the
  duplicate; an agent never closes an issue by hand.
- `wayfinding` — on a wayfinding map and its tickets, and optional: use it only
  if a person has created it. See [Wayfinding](#wayfinding-maps-and-decision-tickets).

## Hierarchy

For a body of work that splits into several issues, open a parent issue whose
body carries a task list of its children:

```markdown
- [ ] #41 Extract the export service
- [ ] #42 Add CSV export
- [ ] #43 Add the export button
```

GitHub renders a completion count from the ticks. Whether it ticks an item when
its issue closes is unverified, so never rely on it, and never read a tick as
status. Link back to the parent from each child. This is weaker than a real
parent field — accepted deliberately in
[ADR-0002](../decisions/0002-track-work-in-github-only.md).

## Wayfinding: maps and decision tickets

Some efforts are too large to plan in one session. Wayfinding handles them with
a **map**, an issue recording where the effort is going, and a **decision
ticket** for each open question, each answered in a session of its own. These
issues hold decisions, not implementation work: no branch, no pull request, and
never a `Closes` line. The opt-in `wayfinder` skill drives this; the rules below
hold for anyone doing it by hand. [ADR-0004](../decisions/0004-map-wayfinding-onto-github.md)
explains how this differs from the upstream skill it adapts.

**The map** is one issue titled `Map: <destination>`. Its first line reads
"Wayfinding map, not implementation work." It then has these sections:

- **Destination** — the finished state, in a sentence or two.
- **Notes** — background that every session on this map needs.
- **Tickets** — a task list of the child tickets, in the form shown under
  [Hierarchy](#hierarchy). It lets the map link to its children and is not a
  status board: never read a tick as a status.
- **Decisions so far** — one line per resolved ticket: a link to the ticket,
  then a short summary of the answer. The full answer lives on the ticket only.
- **Not yet specified** — questions that are in scope but too fuzzy to ticket.
  Move each one out when it becomes a ticket.
- **Out of scope** — what the effort deliberately leaves out, each with its
  reason and a link to its ticket, if it had one.

**A ticket** is a child issue titled `<Type>: <question>`, where `<Type>` is
`Grilling`, `Research`, `Prototype` or `Task`. The type goes in the title and
never in a label. The body holds, top to bottom:

1. the line "Decision ticket for map #n, not implementation work.";
2. a `Blocked by #a, #b` line, omitted when nothing blocks the ticket;
3. a `## Question` section, small enough for one session.

**Chart in two passes.** File the map and the tickets first. Once they have real
numbers, edit in the `Blocked by` lines and the map's task list. A number
guessed beforehand points at some unrelated issue.

**Resolved is not closed.** A ticket is **resolved** when its answer is posted
on it as a comment beginning "Resolution:" and a line for it is in **Decisions
so far**. A resolved ticket stays open, and a person closes it, as they close a
`duplicate`. A person closes the map too. Neither the map nor any ticket is
ever named in a `Closes` line; if the destination is a change to the
repository, it is built through ordinary issues and pull requests, each closing
only itself.

**The frontier** is what a session may pick up next. Derive it from the
`Blocked by` lines and **Decisions so far**; task-list ticks and labels play no
part, and the design works even if GitHub never ticks a task list. A ticket is
on the frontier when it is open, not under **Out of scope**, unclaimed, and each
of its blockers is cleared. A blocker is **cleared** when it appears in
**Decisions so far**, appears under **Out of scope**, or has been closed by a
person. Grilling, Prototype and Task tickets on the frontier still wait for the
requester to start them; only Research tickets may be taken without that.

**Claim by comment.** Before starting, read the ticket's comments, then comment
"Claimed." If you stop without resolving it, comment "Released." Whichever of
the two came last governs, so a claim that was released no longer counts. After
claiming, read the comments again; if another claim that still counts is earlier
than yours, yours does not stand. A person may free a claim
whose session has died by posting "Released." themselves. Do not use the
assignee field: an agent may be acting through the requester's account, and
then the assignee cannot tell a session from the person.

**Recording a result.** Comment the resolution on the ticket, add its line to
**Decisions so far**, and turn anything in **Not yet specified** that the answer
made concrete into tickets. To rule a ticket out of scope, comment on it and add
a line under **Out of scope**; do not close it. Sessions may run at once, and
`gh issue edit --body` overwrites the entire body. So read the map just before
you edit it, change only your own lines, and read it again to check yours
survived.

**No branches.** Wayfinding makes no branch and no pull request, so the branch
rule under [Identity](#identity) does not apply. Research only reads. A
prototype that needs code runs in a detached worktree
(`git worktree add --detach`) or in a scratch directory outside the repository,
is never pushed, and reports what it learned as text on the ticket. If it is
worth keeping, file an ordinary issue for it.

**Other issue formats do not apply.** A map and its tickets do not use the
[product-manager.md](product-manager.md) format, are not refined or sized, and
take no state label except `blocked`. A ticket with an unresolved blocker may
carry `blocked`; whoever resolves the blocker removes the label. If the label
and the `Blocked by` line disagree, the line wins.

**Not for implementation.** An agent asked to implement a map or a decision
ticket stops — see **Prepare** in [implementation.md](implementation.md).

Everything above is creating an issue, editing a body or labels, or commenting,
all pre-authorized under **Authorization** below. None of it needs `gh api` or
a native sub-issue or dependency feature.

## Estimation

There is no story-point field and no velocity. Refinement still produces a
relative size — XS / S / M / L / XL — and it belongs in the refinement comment as
a sizing judgement, not as a metric anything totals.

Anything that comes out **L or XL is a split**, not an item. Refinement proposes
the split and the product manager files the pieces.

## Comments

Anything a stage produces that a reviewer needs — the refinement assessment, a
UX design handoff, the engineer's technical approach for substantial work,
escalation signals, a code review — is posted as a comment on the issue or the
pull request, whichever it scopes.

A decision that constrains future work is promoted to an ADR under
[docs/decisions/](../decisions/); a constraint that recurs across issues is
promoted to a rule in `/docs`. Issues close and are read only by whoever goes
looking; those two do not.

## Authorization

Creating and updating issues, commenting on issues and pull requests, marking a
verified pull request ready for review, and merging a reviewed one are
pre-authorized — see **How far an agent goes** in [pipeline.md](pipeline.md).
No stage asks permission to record its own output. Reading needs no approval
either.

What is still never an agent's to do: closing an issue by hand, deleting an
issue or a comment, and merging a pull request without a posted review.
