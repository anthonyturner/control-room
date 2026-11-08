# Domain modeling workflow

Domain modeling is a grilling session that also keeps a written **glossary**: one
agreed name for each concept the project talks about, so people, issues and
code use the same words. When a hard choice comes up during the session it can
also offer to write a decision record.

**It is opt-in.** It is the `grill-with-docs` skill, and it runs only when the
requester asks for it by name — see [rule 13](../rules.md). It is a grilling
session with extra duties, so everything in [grilling.md](grilling.md) applies
first: rounds, the frontier, facts found by you and decisions made by the
requester. This file adds only what is different.

The technique is adapted from
[mattpocock/skills](https://github.com/mattpocock/skills) (MIT),
`skills/engineering/grill-with-docs` and `skills/engineering/domain-modeling`,
restated here in the playbook's words with a project addendum appended. The
licence notice is in
[third-party-notices.md](../third-party-notices.md).

## The technique

Four habits run through the whole interview, and two duties follow from them.

1. **Compare every name with the glossary.** If the requester uses a word the
   glossary already defines as something else, stop there and ask which
   meaning they want.
2. **Split overloaded words, merge duplicates.** One word covering two ideas
   ("account" as the login, or as the billing arrangement) becomes two terms;
   two words for one idea become one. Propose the names and let the requester
   choose.
3. **Probe the seams.** Where two concepts meet, make up a concrete awkward
   case (an order refunded after its customer was deleted) and ask what ought
   to happen. A specific case exposes gaps that a general question lets slide.
4. **Test claims against the code.** What the requester says the system does is
   checkable, so read the code before accepting it. Put any mismatch to them
   and ask which side is wrong.

The duties:

- **Record a term the moment it settles,** not at the end. The glossary holds
  names and meanings only: no design, no spec, no working notes.
- **Offer a decision record rarely.** Offer one only for a choice that is hard
  to reverse, that a future reader would find puzzling without the backstory,
  and that was picked over real alternatives. Two of the three is not enough.

## The glossary

The glossary is `docs/glossary.md`. If the project already keeps one somewhere
else (a root `GLOSSARY.md`, say), use that one; never start a second. Do not
create the file before the first term settles, and never leave it empty.

Layout: a title with a line on what the project is for, then a `Language`
section. Each entry is the term in bold, what it is in at most two sentences,
and an `_Avoid_:` line naming the rejected synonyms. Add subheadings only if
the entries fall into obvious clusters.

Keep it honest with three rules:

- **One name per concept.** Pick the best word; every other word people use for
  it goes on the `_Avoid_` line.
- **Say what it is, not how it behaves.** Behaviour belongs in the code and in
  specs; an entry that explains mechanics has outgrown the glossary.
- **Only the project's own vocabulary.** A word any programmer already knows
  (retry, cache, feature flag, DTO, log level) stays out, however much of the
  codebase leans on it.

This version keeps one glossary. A project that needs separate vocabularies for
separate contexts is out of scope.

## Project addendum

**When it runs.** Only when the requester asks by name. Like grilling, it is
never a gate: an issue drafted without it is a normal issue.

**Where it runs.** In the main conversation with the requester, never in a
subagent, for the same reason as [grilling.md](grilling.md).

**What it may write.** Exactly two things, and nothing else:

1. The glossary, created when the first term settles and edited as terms settle.
2. After the requester says yes to an offer, one decision record: a new file in
   `docs/decisions/` plus its row in the index there. How to write one — when it
   is warranted, the numbering, the template, the status values — is in
   [decisions/README.md](../decisions/README.md), which stays authoritative.
   The three-part test above only decides when to _offer_.

No other file, no branch, no commit, no issue. A fresh install contains no
glossary; it appears only because a session produced one.

**The writes are uncommitted.** The session edits the requester's checkout in
place. This is a deliberate, requester-invoked exception to the rule in
[implementation.md](implementation.md) and [pipeline.md](pipeline.md) that
nothing is written into the default branch's working tree: the requester asked
for a session that writes, and a branch would need an issue the session may
never produce.

An uncommitted edit that nobody picks up can be lost when the tree is cleaned,
and can make the next `git pull` fail once the same file lands on the default
branch. So the session closes by:

1. Listing every file it wrote or changed, in the closing summary.
2. Recommending to the requester that those files go in through a small docs
   issue and `open-pr` ([skills.md](skills.md)). The recommendation is all the
   session does about it: it files no issue and opens no pull request, and acts
   only if the requester asks.

**When the environment will not let the session write** (a read-only sandbox,
for example), say so, keep the entries in the conversation, and put them in the
closing summary as text the requester can save.

**Ungrillable questions, defects, question format, one at a time.** As in
[grilling.md](grilling.md).

**Using the glossary later.** Once `docs/glossary.md` exists it is shared
reference: any stage may look a term up in it. Only a session the requester
started may change an entry. A stage that thinks a term is wrong says so in the
issue or pull request instead.
