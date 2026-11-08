# ADR-0004: Map wayfinding onto GitHub

- **Status:** Accepted
- **Date:** Accepted when this playbook was installed
- **Deciders:** Project maintainers, by adopting the agent playbook
- **Related:** [tracking.md](../agent-workflows/tracking.md), [rules.md](../rules.md) (rules 3, 12, 13, 14), [ADR-0002](0002-track-work-in-github-only.md), [ADR-0003](0003-agent-autonomy.md)

This narrows [ADR-0002](0002-track-work-in-github-only.md) in two respects: a
wayfinding issue is closed by a person, not by a merged pull request, and it
has a **resolved** state that is not closed. It does not supersede it.

## Context

Wayfinding is a way to plan work too big for one session. One issue is the
**map**: where the effort is heading, what has been decided, and what is still
unknown. Each open question is a child **ticket**, resolved one session at a
time. The upstream `wayfinder` skill (`mattpocock/skills`, MIT) defines the
technique, and the playbook offers it as an opt-in skill.

Upstream was written for a tracker with different rules from ours. Five of its
mechanisms collide with what this playbook already fixes:

- **It closes tickets itself.** Closing a resolved ticket is how it records the
  answer, and "every blocker is closed" is how it works out which tickets can be
  taken next. [Rule 12](../rules.md) says no agent closes an issue, the settings
  deny `gh issue close`, and [ADR-0002](0002-track-work-in-github-only.md) says
  every tracked issue is closed by a merged pull request carrying `Closes #n`.
  A decision ticket has no pull request.
- **It invents labels.** It requires `wayfinder:map` and `wayfinder:<type>`.
  [tracking.md](../agent-workflows/tracking.md) fixes a label vocabulary and says
  never to invent one.
- **It uses native GitHub relations.** Parent/child and blocking go through the
  tracker's own sub-issue and dependency features, which need `gh api`. Our
  hierarchy is a parent task list, accepted as weaker in ADR-0002, and `gh api`
  is not pre-authorized.
- **It claims by assignment.** An agent often acts through the requester's own
  GitHub account, in which case an assignee cannot tell "a session holds this"
  from "the requester does".
- **It pushes throwaway branches.** Research is captured on a `research/<name>`
  branch that is pushed and never merged. Rule 3 forbids deleting branches, so
  these would pile up for good, and the branch rule in tracking.md ties a branch
  to an issue's implementation.

Upstream's lifecycle also assumes a ticket's state is its open/closed state.
Here, closing is a person's act, so the playbook needs its own definition of
"done" for a ticket.

## Decision

We will run wayfinding on the mechanisms the playbook already has, and change no
hard stop to fit it.

- **Resolved is not closed.** A ticket is resolved when its answer is posted on
  it and a line for it is in the map's "Decisions so far". It stays open. A
  person closes resolved tickets and the map, just as a person already closes a
  `duplicate`. No map or ticket is named in a `Closes` line; a destination that
  is a change to the repository is built through ordinary issues and pull
  requests, each closing only itself.
- **The frontier comes from the map and the tickets' text.** A ticket is takeable
  when its `Blocked by #n` lines point only at tickets that are in "Decisions so
  far" or "Out of scope", or that a person has closed. Neither task-list ticks
  nor labels decide it, so the design holds even if GitHub never ticks a task
  list.
- **No new required labels.** The type of a ticket is a title prefix:
  `Grilling:`, `Research:`, `Prototype:`, `Task:`. One optional `wayfinding`
  label joins the vocabulary and is used only if a person creates it.
- **Existing hierarchy, no native relations.** The map holds a task list of its
  tickets. Blocking is a `Blocked by #n` line, with the `blocked` label as a
  convenience. Every write is creating an issue, editing a body or labels, or
  commenting.
- **Claim and release by comment**, not assignment. A person may release a claim
  whose session has died by posting the release themselves.
- **No branches.** Research is read-only. A prototype runs in a detached
  worktree or scratch directory, is never pushed, and reports on the ticket.
- **Not implementation work.** Maps and tickets say so on their first line,
  [implementation.md](../agent-workflows/implementation.md) stops on that
  line, and the product-manager issue format does not apply to them.
- **Starting a grilling, prototype or task ticket is the requester's act.**
  Only research tickets may be taken without the requester starting them; they
  change nothing and need no person, so [rule 14](../rules.md) applies to them.
  The wording of [rule 13](../rules.md) follows when the skill ships.

## Alternatives considered

- **Take upstream as written, and relax the rules for it.** Agents close
  resolved tickets, `wayfinder:*` labels, native sub-issues and dependencies,
  pushed `research/*` branches. Rejected: it weakens two hard stops that the
  settings enforce (closing issues, deleting branches), runs two hierarchy
  models side by side, and makes `gh api` routine. Every project pays for a
  skill most will never switch on. A future refresh of the upstream text could
  bring any of this back, which is why it is written down here.
- **Let agents close resolved tickets as a narrow exception.** Rejected for the
  same reason: "never close an issue" is useful because it has no exception, and
  the cost it avoids, one person closing tickets, is small.
- **Close tickets in a batch through a pull request** with `Closes #...` lines
  for each resolved ticket. Rejected: a map whose destination is a spec or a
  decision ends with no pull request at all, so there is nothing to carry the
  lines.
- **A map with no child issues**, tickets as checklist lines and sessions
  commenting on the map. This removes every tension above, but loses a thread
  per ticket, claiming, and parallel research. A single thread would also grow
  to dozens of sessions. Rejected; reconsider if the child-issue cost proves too
  high.
- **Native sub-issues and dependencies for wayfinding only.** Rejected here
  because two hierarchy models cost more than they give. If native relations
  are wanted, adopt them playbook-wide in an ADR that supersedes
  [ADR-0002](0002-track-work-in-github-only.md).

## Consequences

- **What this makes easier.** Wayfinding needs no new permission, label, branch
  rule or tool. Rules 3 and 12 keep meaning what they say. A project that never
  uses it pays nothing beyond reading this record.

- **What this makes harder, and what it costs.**
  - **A person closes every ticket.** A large map may have dozens. This is cheap
    only if GitHub's issue list closes several at once, which has not been
    checked.
  - **Resolved tickets look open.** The issue list cannot tell a resolved
    ticket from a waiting one. The map's "Decisions so far" is the only place
    that does.
  - **No native view of the frontier.** GitHub shows no blocked-by graph for our
    text lines; the frontier is read from the map.
  - **Edits to the map can race.** `gh issue edit --body` replaces the whole
    body, so two sessions editing together can lose a change. Re-reading before
    and after an edit reduces that, and does not remove it.
  - **A claim is a comment, not a lock.** Two sessions can claim together; the
    earlier claim that has not been released wins, and a session re-reads to
    find out.
  - **Task-list ticks are unverified.** GitHub may or may not tick a task-list
    item when its issue closes, and nobody has checked. The playbook never reads
    a tick, so the map works either way; the completion count may be wrong.

- **What now has to be true.** No wayfinding instruction or wrapper closes an
  issue, deletes an issue or comment, creates a branch, or pushes. Maps and
  tickets stay recognizable as "not implementation work" so that an agent does
  not branch and open a pull request for one. A change that wants any of the
  rejected mechanisms supersedes this ADR first.

## Compliance

Read the wayfinding section of [tracking.md](../agent-workflows/tracking.md) and
any wayfinding skill or workflow document for these things, each of which is a
finding: an instruction to close or delete anything; a branch, push or pull
request for a map or ticket; a `wayfinder:` label or any label other than
`wayfinding` and `blocked`; `gh api`, a native sub-issue or a native dependency;
a frontier worked out from labels or task-list ticks; or an assignee used as the
claim. In a live map, every ticket a session has resolved is still open, and no
branch was created.
