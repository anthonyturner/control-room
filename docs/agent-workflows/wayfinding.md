# Wayfinding workflow

Wayfinding plans work that is too big for one session. You cannot yet see the
whole way from the idea to the finished thing, so you do not start building: you
chart it first. One issue, the **map**, says where the effort is going. Every
open question on the way becomes a **decision ticket**, an issue of its own that
one session settles. When the questions run out, the route is clear enough to
hand to the normal pipeline.

**It is opt-in.** It is the `wayfinder` skill, and it runs only when the
requester asks for it by name. It is not a gate, not a precondition for filing
an issue, and never something an agent starts on its own initiative — see
[rule 13](../rules.md).

The technique is adapted from
[mattpocock/skills](https://github.com/mattpocock/skills) (MIT),
`skills/engineering/wayfinder`, restated here in the playbook's words with a
project addendum appended; the licence notice is in
[third-party-notices.md](../third-party-notices.md).
Where this playbook differs from the upstream skill, and why, is recorded in
[ADR-0004](../decisions/0004-map-wayfinding-onto-github.md).

How a map and its tickets are written down in GitHub — their shapes, the
frontier, claiming, labels, and what "resolved" and "closed" mean — is
**Wayfinding** in [tracking.md](tracking.md). This file holds the technique and
does not repeat those rules.

## When to use which

Pick by how far the work reaches, not by how important it is:

| The work… | Use |
| --- | --- |
| is small enough to check afterwards | Just do it, through the normal pipeline ([pipeline.md](pipeline.md) says when the full one runs). No skill. |
| fits in one session but needs thinking | `grill-me` or `grill-with-docs` ([grilling.md](grilling.md)), then `plan-issue`. |
| spans several sessions, with fog ahead | `wayfinder`. |

A map ends in one of three places. Each has an exit into the normal flow, and
wayfinding never does the work of the exit itself:

| The destination is… | The map hands over by… |
| --- | --- |
| a spec | Giving the map's Destination and Decisions so far to `plan-issue` as the request. Refinement still runs, and anything that comes out L or XL is still split. |
| a decision | Filing an issue for the record and landing an ADR through the usual issue and pull request ([decisions/README.md](../decisions/README.md)). |
| a change made in place | `plan-issue`, then `implement-issue`. The map decides what to change; the pipeline changes it. |

Whatever the exit, it also carries the terms and decision-record candidates that
grilling tickets settled along the way. The glossary entries and ADRs are
written there, in that issue's pull request, never during wayfinding.

Wayfinding is never a precondition for `plan-issue`. An issue drafted without a
map is a normal issue, and no stage may refuse to proceed for want of one. If
charting finds that the way is already clear, that is a result, not a failure:
the work did not need a map.

## A worked example

The rest of this file uses one example. A project wants customers to export
their data.

- The map is `Map: Customers can export their data`. Its destination is a spec.
- Its tickets, each titled `<Type>: <question>`:
  - `Grilling: Which formats do customers need?`
  - `Research: What size limit does the storage provider put on a download?`
  - `Prototype: Where does the export button sit on the settings page?`
  - `Task: Request access to the staging bucket`
- Its fog, in **Not yet specified**: who may export. Nobody can yet word that as
  a sharp question.

Once the formats ticket is resolved as "CSV for everyone, JSON for admins", the
fog sharpens and a new ticket appears: `Grilling: Which roles count as admins?`

## The technique

**Decide first, build later.** Test the map with one question: could someone open
the first pull request tomorrow without having to ask anybody anything? Until the
answer is yes, a question is still open. If you feel the pull to start building
early, you are either at the map's edge, so hand over, or you are skipping a
question, so write it down. A map's **Notes** can ask a session for more than a
bare answer, such as an outline drafted in the resolution comment. They cannot
authorize a repository edit, a branch, a commit, a push or a pull request. A
change made in place leaves through `plan-issue` and `implement-issue`.

**Say titles, not numbers.** A report that reads "#42 is blocked by #43 and #44"
sends the reader off to look each one up. Name a ticket by its title instead,
written as the text of its link: "[Which formats do customers need?](url)
unblocks [Which roles count as admins?](url)". Do this in narration and in
**Decisions so far**. The number and URL stay inside the link; they never
replace the title.

**One home per fact.** The ticket holds the whole answer. The map holds a
one-line gist and a link. Where the two ever disagree, the ticket is right and
the map line is the thing to fix.

**Ticket or fog?** Ask whether you can state the question as one sentence that a
single session could answer.

- If you can, it is a **ticket**, even when something else blocks it. "Which
  roles count as admins?" is a ticket the moment it can be worded that way.
- If you cannot, it is **fog**: a line under **Not yet specified**, as vague as
  it needs to be. "Who may export" stays fog until the formats are settled.

Do not break fog into tickets in advance. Settling one question often turns a
single patch of fog into three tickets, or into none. After each resolution,
reread the fog and promote whatever has become sharp.

**Out of scope is not fog.** The destination draws the boundary. Something
beyond it goes under **Out of scope** with its reason, and comes back only if the
destination is redrawn, as a new effort. If an existing ticket turns out to lie
past the boundary, rule it out as [tracking.md](tracking.md) describes; a
ticket ruled out this way no longer holds up anything it blocked. Ruled-out work
never goes in **Decisions so far**, which records only the route actually
walked.

**One question, one session.** Word each ticket so a single session can settle
it without running so long that it needs a fresh start. A question that would
run longer is two tickets. Only research is settled in bulk: several research
tickets can go in parallel, because none needs a person. Other sessions may be
editing the same map at the same time, so follow the edit discipline in
[tracking.md](tracking.md).

## Ticket types

The type is the title prefix: `Grilling:`, `Research:`, `Prototype:` or `Task:`.
Read it from the title; the body states only the question.

| Type | Who is involved | What the session does |
| --- | --- | --- |
| Grilling | Person in the loop | Interviews the requester until the question is answered. |
| Research | Agent alone | Reads outside material and reports a fact. |
| Prototype | Person in the loop | Builds something rough for the requester to react to. |
| Task | Person in the loop | Does, or hands over, manual work that unblocks a decision. |

An agent never answers the requester's side of a person-in-the-loop ticket for
them.

**Grilling** is the default type and the only one that is a conversation. Run
the technique in [grilling.md](grilling.md). When the requester wants the
vocabulary pinned down too, use the habits from
[domain-modeling.md](domain-modeling.md) (compare each name with the glossary,
split overloaded words, probe the seams, test claims against the code) but
**write no file**. The terms that settle go in the resolution comment, each with
its meaning and the words to avoid. A choice worth a decision record is named
there as a candidate. Both reach the repository only through the exit's issue
and pull request. For the example above, the roles ticket might end with
"**Admin**: a user who can manage billing. _Avoid_: owner, superuser".
`grill-me` writes nothing at all, so posting the outcome on the ticket is this
workflow's job and comes after the interview.

**Research** answers a question of fact from material outside the repository:
documentation, a third-party API, a vendor's published limits. It is
**read-only**. It changes nothing in the repository, creates no branch, and
reports on the ticket as text. A research ticket needs no person, so it follows
[rule 14](../rules.md): decide, and carry on. A read-only subagent may do it. If
that subagent cannot reach GitHub, the caller posts what it found, as **The
caller posts what read-only stages cannot** in [planning.md](planning.md)
describes. If the research needs the web and the subagent has no network, run
it in the main session.

**Prototype** is for questions the requester can only answer by seeing or
touching something, such as where the export button sits. Make the cheapest
thing that shows it: a sketch, a stub, an outline, a few lines of throwaway
code. Keep it off the repository's branches. Work in a scratch directory away
from the repository, or in a detached worktree when the repository's own code is
needed. Do not commit it or push it, and remove it once the ticket is resolved.
The ticket records what the requester learned, as text. If the prototype is worth
keeping, it becomes an ordinary issue and pull request.

**Task** is the one type that does something instead of deciding something, and
it exists only to unblock a decision. Its case is manual work with nothing to
decide, research or prototype: requesting access to the staging bucket so the
size limits can be tested, or importing a handful of sample records to learn
which fields they carry. The agent does only what [pipeline.md](pipeline.md)
already authorizes. Whatever
else is needed it hands to the requester as a short, exact checklist, and the
requester reports back. The resolution records what happened and the facts later
tickets need (where a credential is kept, a new URL, a row count). It never
contains the secret itself.

## Charting a map

The requester brings a loose idea. A charting session:

1. **Fixes the destination.** Interview the requester ([grilling.md](grilling.md))
   until it is clear whether the map finds its way to a spec, a decision or a
   change in place. Do this first, because the destination sets the scope.
2. **Surveys the whole space before going deep.** Ask wide questions: what has to
   be decided, in what order, what is already known, what could be started today.
   The aim is a list of open questions and the first few that can begin now, not
   an answer to any of them. **If the survey finds nothing open**, because the
   way is already clear and the journey fits one session, stop and ask the
   requester how to proceed. That stop is deliberate; do not decide it for them.
3. **Creates the map**, with Destination and Notes filled in, **Decisions so
   far** empty, and the fog sketched under **Not yet specified**.
4. **Creates every ticket it can word sharply now.** It then adds the `Blocked
   by` lines and the map's task list in a second pass, once every ticket has its
   real number. [tracking.md](tracking.md) explains why.
5. **Settles the research tickets that are ready**, in parallel, each in its own
   session or subagent. Research needs no person, so [rule 14](../rules.md)
   applies.
6. **Stops.** Charting is one session's work, and it resolves nothing except
   research.

## Working through a map

The requester gives a map and may name a ticket. If none is named, the session
chooses one itself.

1. **Load the map** alone: the low-resolution view, not every ticket.
2. **Choose a ticket.** Use the one named. Otherwise take the first ticket on the
   frontier, in the order of the map's task list; **Wayfinding** in
   [tracking.md](tracking.md) says what the frontier is. A ticket that already
   carries a claim is not yours. If that claim looks dead, say so to the
   requester: only a person frees it. A ticket that needs the requester
   (anything but research) is for a session where the requester is present. With
   nobody there to answer, take a research ticket or stop.
3. **Claim it** by comment, before any work ([tracking.md](tracking.md)).
4. **Resolve it** as its type says, above. Fetch the full body of any related
   ticket when you need it, and read whatever the map's Notes tell every session
   to consult.
5. **Record the resolution** on the ticket and in the map, and leave the ticket
   open ([tracking.md](tracking.md)). Resolved is not closed: a person closes
   every resolved ticket, and the map.
6. **Move the fog.** Create and wire the tickets this answer made specific, and
   take each one's patch out of **Not yet specified**, so it lives only as its
   ticket. If the answer puts a ticket beyond the destination, rule it out of
   scope instead of resolving it. If it makes other tickets wrong, edit them to
   match, or rule them out with a comment that says why. Never delete a ticket.
7. **Report** what was settled, what is now on the frontier, and which resolved
   tickets are waiting for a person to close them.

The map is complete when every ticket is either resolved or ruled out of scope,
**Not yet specified** is empty, and no ticket is still claimed. An empty frontier
is not enough: a claimed, unresolved ticket is off the frontier but not done. Say
so, and name the exit that matches the destination. A person closes
the map, as they close its tickets; the pull request that finally builds
something never does, because it names only its own issue.

## Project addendum

**When it runs.** Only when the requester asks by name. A requester who invokes
the `wayfinder` skill on a map starts the grilling, prototype and task tickets
it works in that session. No agent, and no unattended or looping run, starts one
of those on its own. Research tickets follow [rule 14](../rules.md).

**Where it runs.** In the main conversation with the requester. A subagent has no
channel to them, so only research is ever delegated.

**What it writes.** Only GitHub issues: creating them, editing a body or labels,
and commenting, which are all pre-authorized ([tracking.md](tracking.md)). The
labels it may use are the ones listed there. No repository file, no branch, no
commit, no push and no pull request comes out of a wayfinding session, however
the session was started and whatever the map's Notes say. That includes the
glossary and decision records, which grilling tickets hand to the exit as text.
The one place code may be touched is a prototype's throwaway work, in a scratch
directory or a detached worktree, as described under **Prototype**. It is never
committed or pushed, and it is not a change to the repository.

**Where it keeps the map.** In GitHub, always. The upstream skill can fall back
to local markdown files when it has no tracker. This playbook has a tracker
([rule 11](../rules.md)), so that fallback is not used and nothing about a map is
written to a file.

**Defects.** A wayfinding session is not how a defect is diagnosed. Establishing
the cause is engineering's job under **Defect lane** in [pipeline.md](pipeline.md).

**Question format.** Follow [../response-style.md](../response-style.md) in
every question and report: plain English first, technical detail underneath, and
a repository-specific name explained the first time it appears.
