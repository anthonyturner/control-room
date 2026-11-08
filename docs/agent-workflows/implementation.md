# Implementation workflow

Use this workflow to implement a filed work item. The item is tracked as a
GitHub issue — read [tracking.md](tracking.md) first. The issue is the scope
boundary; an issue design comment is authoritative for UI/UX when the optional
UX design stage was used. Architecture, interfaces, data flow, and technology
choices are decided here, as part of implementation, not by the UX design
stage — this workflow owns the technical solution.

Write the technical approach, the change report, and the pull-request body for
the requester, following the response formatting and audience rules in
[../response-style.md](../response-style.md): plain English first, technical
detail underneath, jargon explained the first time it appears, and file and
line references kept as evidence.

## Prepare

1. Read `AGENTS.md`, [../rules.md](../rules.md), [tracking.md](tracking.md),
   the complete work item — the GitHub issue and all of its comments — any
   design comment, and every relevant `/docs` instruction file, including the
   files under `docs/stack/`.
   Read `.github/copilot-instructions.md` too.
2. Stop if the issue's first line says "not implementation work". It is a
   wayfinding map or decision ticket, which holds decisions and gets no branch
   or pull request. See **Wayfinding** in [tracking.md](tracking.md). Say so
   and do nothing further.
3. Confirm GitHub authentication, repository identity, default branch, remotes
   and the primary checkout's `git status --short`, all read-only.
4. Leave whatever is in the primary checkout alone, however much there is
   ([rule 5](../rules.md)). The work happens in a worktree, below.
5. Use a branch named `<type>/<issue-number>-<short-description>` based on the
   current `main`, e.g. `feat/123-saved-filters`. The type is
   a Conventional Commit type, per [tracking.md](tracking.md).

## Work in a worktree, never the primary checkout

Implementation **always** runs in a worktree: a second checkout of the same
repository, on its own branch, under
`.claude/worktrees/<issue-number>-<short-description>`. The state of the primary
checkout does not matter: clean, dirty or mid-edit, the answer is the same.

The primary checkout is the one the requester has open. Switching its branch
changes the files under their editor, and anything uncommitted there is theirs
([rule 5](../rules.md)). A worktree lets them keep working while you work, and
lets agents run in parallel without sharing a tree.

In the primary checkout an agent runs only:

- read-only git, for example `git status`, `git diff`, `git log`, `git show`,
  `git branch --show-current`, `git check-ignore`, `git rev-parse`;
- `git fetch`;
- `git worktree add`, `git worktree list`, `git worktree move` and
  `git worktree remove`.

Never `git switch` or `git checkout` a branch there, and never commit, pull,
merge, rebase, reset or stash there. Never edit its files unless the requester
explicitly asks.

```sh
git check-ignore -q .claude/worktrees/x ||
  echo '.claude/worktrees/' >> "$(git rev-parse --git-common-dir)/info/exclude"
git fetch origin
git worktree add .claude/worktrees/<issue-number>-<short-description> \
  -b <type>/<issue-number>-<short-description> origin/main
```

The first line keeps the folder out of `git status`. Without it the primary
checkout shows `?? .claude/worktrees/` from the first worktree on, and a careless
`git add` stages a whole checkout. It writes to the repository's local exclude
file, so nothing is committed.

- Use `.claude/worktrees/` — one known location keeps them findable and easy to
  ignore.
- Run every command with the worktree as the working directory. Before each
  commit, `git branch --show-current` must print the feature branch.
- Install dependencies inside the worktree with `npm ci`. Never
  share or link another tree's installed dependencies into it: a shared
  install means one tree's change rewrites every tree's dependencies at once.
- If the install fails because the lockfile no longer matches the dependency
  manifest, the branch is stale, not broken: rebase it on
  `main`. Do not regenerate the lockfile to make the error go
  away — that is a dependency change nobody asked for ([rule 6](../rules.md)).
- Gitignored local files the build needs, such as local environment settings,
  do not exist in a new worktree. Copy them in, and confirm with
  `git check-ignore` that they cannot be committed.

A worktree is scaffolding for one issue. Remove it once its pull request
merges (below) rather than leaving it to be reused by a later task, where it
drifts out of step with the primary checkout.

### The folder name binds on every creator

`.claude/worktrees/<issue-number>-<short-description>` is not only a convention
for people typing `git worktree add`. It binds on **anything** that creates a
worktree here, including a coding agent's own worktree-isolation feature, which
may name the folder after the agent's session (`agent-<id>`). A name like that
says nothing about what is inside it, and these accumulate alongside the
correctly named ones.

Two ways to comply:

- **Do not use the agent's auto-naming isolation.** Create the worktree with the
  command above instead, which names it correctly to begin with.
- **Or rename it before the first edit:**

  ```sh
  git worktree move .claude/worktrees/agent-<id> \
    .claude/worktrees/<issue-number>-<short-description>
  ```

  Rename before opening an editor on it, never after. On some systems the move
  fails outright while any program holds the folder open.

**Why the issue number and not the pull-request number.** The order this
document requires is worktree → branch → push → draft pull request → implement.
When the folder is created the pull request does not exist yet; only the issue
number does. Naming by pull request would need a rename afterwards, and would
invalidate any path already stored or any editor already open on it. The issue
number exists before the worktree does and never changes.

**A folder name is never evidence of what a worktree holds.** It is set once, at
creation, and a branch can be renamed afterwards — a folder named
`40-export-button` can hold branch `fix/41-export-encoding` for pull request 45.
So **match on the branch, never on the folder name**, when resolving which
worktree holds a piece of work. `git worktree list` prints the branch beside
each path, which is the reliable half. Opening the wrong one means editing the
wrong branch with no warning that you have.

### Clear the originals when the pull request opens

Copying files into a worktree leaves the originals sitting in the primary tree.
Once the pull request merges they become byte-identical duplicates of what is
now on `main`, and git refuses to overwrite them — so the
requester's next `git pull` fails:

```text
error: Your local changes to the following files would be overwritten by merge
error: The following untracked working tree files would be overwritten by merge
```

Untracked files are the harsher half: git will not clobber one even to create
an identical copy, so a new file blocks the pull just as firmly as an edited
one.

Tell the requester to clear the originals when the pull request opens, not when
it merges. By then the content is committed and pushed, so nothing is at risk,
and the requester is still in the conversation. Clearing means `git restore` or
deleting files in the primary checkout, which is not an agent operation: the
requester does it, or asks for it as an explicit request.

Confirm the originals are unchanged before saying they are safe to discard:

```sh
git diff --stat origin/<branch> -- <paths>                             # tracked files
git show "origin/<branch>:<path>" | diff - <path>                      # an untracked file
git ls-tree -r --name-only origin/<branch> -- <path>                   # is it in the commit at all
```

If they differ, the requester edited them after the copy. Show them the
difference rather than calling the originals safe to discard. These are their
changes, not yours.

Never read a failing `git show` as evidence that work did not merge; some
shells rewrite a `<rev>:<path>` argument before git sees it. Confirm a merge
from the pull request.

### Remove the worktree once the pull request is merged

A worktree is scaffolding for one issue, not a permanent checkout. Once the
pull request has merged, remove it. Run the command from the primary checkout,
not from inside the worktree: a worktree cannot remove the folder it is standing
in, and the relative path below resolves from the primary checkout.

```sh
git worktree remove .claude/worktrees/<issue-number>-<short-description>
```

**Determine "merged" from the pull request, never from the commit graph.**
`gh pr view <n> --json state,mergedAt` is the authority. `git merge-base
--is-ancestor <branch> origin/main` is not: a squash or
rebase merge rewrites the commits, so a fully merged branch stops being an
ancestor of the default branch. Trusting ancestry leaves finished worktrees
behind in one repository and deletes live work in another.

Never remove a worktree for an open pull request, and never on the assumption
that a merge happened.

**Sweep at the start of every session; do not wait to notice.** With the
plugin installed in Claude Code, a session-start hook lists the worktrees whose
pull request has merged, split into clean ones to remove and ones with
uncommitted changes to leave alone. Remove the clean ones before other work.
Without the hook, run the same check yourself when a session starts:
`git worktree list`, then the pull request state of each worktree under
`.claude/worktrees/` — by its branch, or for a detached one, such as a review
checkout, by whether its `HEAD` is a merged pull request's head commit
(`headRefOid`) or already on the default branch. A detached checkout has no
branch, so ancestry is safe there: a `HEAD` on the default branch holds no
commit a removal could lose. The checks below apply to every removal.

Check the worktree is clean first (`git status --porcelain`), and check it
holds no link that points outside it. Uncommitted work inside one is the only
thing a removal can actually destroy — branches and commits survive, because
the branch is never deleted. If anything is uncommitted, leave the worktree
alone and report it.

Removing a worktree is not deleting a branch, which rule 3 forbids. Leave the
branch alone; GitHub disposes of the remote one on merge if the repository is
set to.

The primary checkout never left its branch, so there is nothing to restore
there. It is simply behind `origin/main` until the requester
pulls; an agent does not pull for them.

Abandoned worktrees accumulate. Each one is a full checkout on disk, and
together they make `git worktree list` useless for seeing what is actually in
progress.

## Open the draft pull request before implementing

The branch and the draft pull request go up **before the first edit**, not after
the last one. Nothing is written into the default branch's working tree, the
requester can watch the work arrive, and, where the project has continuous
integration (CI), it runs against every push instead of against one large diff
at the end.

Create the worktree and branch first, as above, then run these in the worktree:

```sh
git commit --allow-empty -m "chore(scope): start <thing>"
git push -u origin <type>/<n>-<short-description>
gh pr create --draft --base main   # body carries Closes #<n>
```

The empty commit is not ceremony: GitHub refuses a pull request whose head
branch is not ahead of base, so a branch with no commits cannot have one. The
scaffold message is still a valid Conventional Commit, so any commit hook
passes it.

Fill the pull-request body in properly as the work lands — summary, verification
evidence, manual-test boundary, risks, escalation signals. A body left as a stub
is an unfinished handoff, not a finished one.

For a **defect**, the cause has to be confirmed before the issue can be filed
honestly (see **Defect lane** in [pipeline.md](pipeline.md)), so the branch
comes first under a provisional name, and both the branch and its worktree
folder are renamed once the issue number exists:

```sh
git branch -m fix/<n>-<short-description>
git worktree move .claude/worktrees/<provisional-name> \
  .claude/worktrees/<n>-<short-description>
```

Run the move from the primary checkout, as **The folder name binds on every
creator** above describes, and before opening an editor on the folder.

## When the work was written before the pull request existed

Sometimes the code already exists in the working tree and no branch or pull
request was ever opened. That is the wrong order — the section above is the
right one — but it happens, and the recovery is not a different workflow. It is
this one, entered late. The `open-pr` skill runs it.

1. Read `git status` and `git diff --stat` in the primary checkout (both
   read-only) and list the files that belong to the issue. Everything else is
   unrelated; leave it exactly where it is ([rule 5](../rules.md)).
2. Resolve the issue the work belongs to (`gh issue view <n>`), and confirm the
   uncommitted changes actually match its scope and acceptance criteria before
   going further. If no issue exists, file one first — see
   [planning.md](planning.md). Untracked work does not get a pull request
   ([rule 11](../rules.md)).
3. Check the primary checkout is not ahead of upstream. Run `git fetch origin`,
   then in the primary checkout:

   ```sh
   git merge-base --is-ancestor HEAD origin/main
   ```

   Passing while the checkout is *behind* `origin/main` is
   normal: agents do not pull for the requester. If the check fails, the
   checkout holds commits that are not on it, for example an unmerged feature
   branch. Stop and report instead of guessing which base the edits belong on
   ([rule 5](../rules.md)).
4. Create the worktree and branch at the primary checkout's `HEAD`, not at
   `origin/main`:

   ```sh
   git check-ignore -q .claude/worktrees/x ||
     echo '.claude/worktrees/' >> "$(git rev-parse --git-common-dir)/info/exclude"
   git worktree add .claude/worktrees/<issue-number>-<short-description> \
     -b <type>/<issue-number>-<short-description> HEAD
   ```

   This **deliberately differs from the normal Prepare step**. The edits were
   made on top of `HEAD`, so a patch reproduces them exactly only on that same
   base. On `origin/main`, upstream changes to a file the work
   also touches would mix into the result, and the comparison in the next step
   could not succeed without overwriting upstream work. Everything else in
   **Work in a worktree** applies, including the dependency install. The
   primary checkout's branch is never switched, and its uncommitted files stay
   where they are.
5. Move the issue's files into the worktree, and only those. Run these from the
   worktree root, because `git apply` resolves paths against the directory it
   runs in:
   - tracked changes, including deletions, renames and mode changes:

     ```sh
     git -C <primary-checkout-root> diff --binary HEAD -- <paths> | git apply --index
     ```

     The patch lands on the same base it was cut from, and `git apply` is
     all-or-nothing: it applies completely or changes nothing. `--index` also
     stages the result, which is what records a deletion or a mode-only change.
     List both the old and the new path of a rename, or the old file stays and
     the new one arrives as a plain addition;
   - new untracked files: `cp -p` them to the same paths. This cannot overwrite
     anything, because a file untracked in the primary checkout is not in
     `HEAD`, and the worktree starts at `HEAD`.

   Then verify nothing was lost: `git status --short` in the worktree lists
   exactly the issue's files, and each one matches its original
   (`diff <original> <copy>`; a deleted file is absent). Do not go on until it
   does.
6. Stage the untracked files by path (never `git add -A` or `git add .`) and
   commit locally, with a Conventional Commit as **Publish** below describes.
   The tracked changes are already staged. Do not push yet, and note the commit
   with `git rev-parse HEAD`.
7. Bring the branch up to date inside the worktree, with the stale-branch
   guidance above: `git fetch origin`, then `git rebase
   origin/main`. The branch is unpublished, so the rebase
   rewrites nothing anyone has. If it conflicts, resolve only what the issue
   makes clear; otherwise stop and report ([rule 5](../rules.md)).
8. Verify, in the worktree, as **Implement and verify** below describes, and
   report pass or fail plainly. Never silently skip a failing check.
9. Summarise the diff for the requester and carry straight on. Do not wait for
   approval ([rule 2](../rules.md)).
10. Push and open the pull request as **Publish** below describes.

The originals stay in the primary checkout. Never delete, revert, stash or
overwrite them unless the requester explicitly asks: they are the requester's,
and **Clear the originals when the pull request opens** above says how they are
handed back. If the rebase in step 7 brought in upstream changes to those files,
the pushed branch rightly differs from the originals there, so run that
section's comparison against the commit from step 6 instead of
`origin/<branch>`. What differs from the normal order is only that the code
existed first; the branch is still created before the commit, and in a worktree
like any other.

## How far this stage goes

There is one mode: **run to the merged pull request without stopping**, per
**How far an agent goes** in [pipeline.md](pipeline.md). Filing the issue,
branching, committing, pushing, opening the PR, reviewing it and merging it
are all pre-authorized. Do not present an approach and wait, and do not present
the final diff and wait — the diff is reviewed on the pull request, in the
open, where the review leaves a record.

Carry any open question into the pull-request body rather than holding the
work back for an answer. When the answer would change what should merge, stop
before merging instead — see **Review and merge** below.

The narrow exception is the requester saying, for a specific piece of work, that
they want to see the diff before it goes up. Honour that for that work; it does
not change the default.

## Technical approach

Decide and state the technical solution before writing code. This is the
engineer's design work, and it is the step the UX design stage deliberately
does not cover.

For a defect, this investigation comes *before* the issue is finalized, not
after: the product manager needs a verified cause to write the issue against.
See **Defect lane** in [pipeline.md](pipeline.md).

1. Identify the root cause, or the mechanism the change hinges on, from the
   code rather than from the issue text alone. For a defect, say what actually
   produces the reported behavior and cite the file and line.
2. Name the module, service, or layer that owns the responsibility, and check
   whether one already does ([rule 10](../rules.md)) before adding a new one.
3. State the approach in a few sentences: what changes, where, why there
   rather than somewhere else, and the alternative rejected with its reason.
4. Note the blast radius — public signatures, existing callers, persisted
   shapes, and anything that stays deliberately unfixed.
5. State the test seams in a line: the public entry point a test will call,
   and the system boundary where a fake may stand in. You choose them; do not
   ask for confirmation ([rule 14](../rules.md)). See [../testing.md](../testing.md).
6. For substantial work, post this as a comment on the GitHub issue before
   implementing, so it is reviewable on its own. For a small or obvious change,
   stating it in the response is enough. Either way it belongs in the
   pull-request body.
7. Report **escalation signals** when they occur, as defined in
   [refinement.md](refinement.md). On this stage the ones that matter most are:
   you revised the root cause after first stating it; you could not reproduce
   the reported behavior; a fix made a test pass without you being able to
   explain why; or two attempts at the diagnosis have not converged. Say so
   plainly instead of proceeding on the third guess — a confident wrong root
   cause is the most expensive failure this pipeline can produce.

## Implement and verify

1. Make only changes required by the issue and approved design.
2. For new logic, write the failing test first, watch it fail, then implement
   against it, and keep the proof that it failed. This is the verification
   method, not an afterthought. How to write the test, what counts as proof and
   which changes are exempt (styling, markup, flows that can only be checked by
   hand in the running app, and the rest) are in [../testing.md](../testing.md).
3. Do not add dependencies or broaden scope without user approval
   ([rule 6](../rules.md)).
4. Run `npm run build` plus `none yet` and `npm test`
   as the change warrants ([rule 8](../rules.md)).
5. Add a line under `## [Unreleased]` in `CHANGELOG.md` if a user of the
   project would notice the change, per [../changelog.md](../changelog.md). If
   not, say so in the pull-request body.
6. Review the complete diff and status for unrelated changes, secrets, and
   debug artifacts.
7. List every changed file for the pull request as a repo-relative markdown
   link with the changed line range(s), e.g.
   `[path/to/file:12-18](path/to/file#L12-L18)` (whole-file link for new
   files), each with a few words on what changed. Keep it tight: group small
   related edits on one line rather than listing every hunk.
8. Write the SOLID note for the pull request. This is the only stage that
   writes one, so the format lives here; [rule 9](../rules.md) is the
   judgement about when SOLID is worth applying at all, and it points at this
   step. Keep it brief: one line per principle applied (how, and why it was
   worth it), and one line for any principle or abstraction deliberately not
   used and why, so a reader can tell a considered omission from an
   oversight.

## Publish

Publication is pre-authorized; the pull request already exists from **Open the
draft pull request before implementing** above. This is where the real content
reaches it.

1. Use a Conventional Commit. Refer to the issue in prose in a commit body
   ("issue 123"), and keep the `#<number>` link for the pull-request body — see
   **Issue references in commits** below.
2. Push without force.
3. Update the pull request to the shape of `.github/PULL_REQUEST_TEMPLATE.md` —
   Summary, Changes, SOLID, Verification, Not done — titled with the
   Conventional Commit subject, with `Closes #<n>` in the body. Replace the
   scaffold body; do not leave a stub. Verification carries the proof of change
   for each new behaviour (or "exception: <reason>"), and the body names the
   test seams, both as set out in [../testing.md](../testing.md).

   Keep the body short (about 200 words) to save output tokens: brief
   bullets, no long rationale, nothing the diff already makes obvious.
4. Report back to the requester briefly: a line or two on the outcome, then
   a clickable link for everything produced — the issue, the pull request,
   and any HTML page, artifact, screenshot or report — and anything that needs
   their decision. Do not repeat what the pull request already says; the
   links are how they reach it. Use short link text followed by the title, with the issue and
   its pull request on one line joined by an arrow, one line per repository:

   ```markdown
   - <repo>: [#123](<issue-url>), <issue title> → [#124](<pr-url>), <pr title>
   ```

Then carry on to **Review and merge**.

## Review and merge

This is the gate that stands where a person reading a draft would otherwise
stand, so it is not optional and not shortened for small changes.

1. **Mark the pull request ready** (`gh pr ready <n>`) once Publish is done and
   every check you ran passes. GitHub cannot merge a draft.
2. **Review it.** Run the review in [qa-review.md](qa-review.md) — the
   `review-pr` skill — against the pull request as it now stands, from a fresh
   read of the diff rather than from memory of writing it.
3. **Post the review on the pull request** as a comment (`gh pr review <n>
   --comment` or `gh pr comment <n>`). Never as an approval: GitHub does not let
   an account approve its own pull request, and the review has to be readable
   by the requester afterwards either way.
4. **Fix the real findings.** Every blocking finding is fixed with a new
   commit on the branch. A non-blocking suggestion is fixed when you agree with
   it; when you do not, reply on the pull request saying why. Push without
   force, then re-review the new diff and post that review too.
5. **Merge when it is clean**, with a merge commit:

   ```sh
   gh pr merge <n> --merge
   ```

   A merge commit keeps the branch's commits intact, so the merge can be
   reverted as one unit and the history shows exactly what was reviewed.

Do **not** merge, and say why in the report and on the pull request, when:

- a blocking finding remains after a second review round;
- a check fails, or a required check has not run;
- merging would need branch protection bypassed (`--admin`) or a conflict
  resolved by guessing which side is right;
- two or more escalation signals were reported on this work;
- an open question in the pull-request body would change what should merge;
- the requester asked to see this piece of work before it merged.

Leave that pull request open and ready for review. Never close the issue by
hand ([rule 12](../rules.md)) — the merge closes it through `Closes #n`.

After merging, remove the worktree per **Remove the worktree once the pull
request is merged** above, and report the merge commit. A pull request left
open keeps its worktree.

## Issue references in commits

Keep `#<number>` for the pull-request title and body, and for a real trailer
line such as `Closes #187`, where GitHub's linking is what actually matters.
In a commit body, spell the reference out in prose (`issue 187`).

This matters more than it looks under commitlint, which this project runs from
`.commitlintrc.json`. Never write a bare `#<number>` anywhere in a commit
**body**: it fails the commit hook, and the error it produces points at the
wrong thing.

commitlint's parser scans the message line by line for issue references
shaped like `#123`. The moment **any** body line contains that pattern, the
parser switches into "footer mode": every line from there on, including
ordinary prose, is reclassified as the footer and checked against footer-only
rules (`footer-max-line-length`, `footer-leading-blank`). It then fails even
when the real trailer further down is perfectly formed.

The error text — typically "footer must have leading blank line" — is
misleading. It is not complaining about the `Co-Authored-By:` or `Closes #n`
trailer. It is complaining about body prose that got swept into the footer by
mistake. It is not about length or line count; wrapped multi-line bodies are
fine. The only trigger is a bare `#<number>` outside the real footer.

Check before committing for real:

```sh
npx commitlint --edit <path-to-message-file> --verbose
```

When it fails, look for a bare `#<number>` in the body first. That is the cause
far more often than anything else.

Never force-push, rewrite history, delete branches, bypass branch protection,
or commit directly to `main`. Never close an issue by hand —
the merge closes it through `Closes #n`.
