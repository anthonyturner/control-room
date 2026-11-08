---
name: implement-issue
description: Implement a tracked control-room GitHub issue through a verified pull request, branching and opening the pull request before the first edit. Use for any request that changes repository files or asks to implement, fix, refactor, document, test, or otherwise complete tracked work. Do not use for filing an issue or producing a design ($plan-issue), for reviewing a pull request ($review-pr), or when the code is already written and only needs a pull request ($open-pr).
---

# Implement an issue (Codex)

Read `AGENTS.md`, `docs/rules.md` and `docs/agent-workflows/implementation.md`
completely and run the workflow implementation.md defines. Those documents are
authoritative; this file adds only what is specific to Codex.

## Codex specifics

- Run the implementation as the `developer` agent from `.codex/agents/`.
- Use the available GitHub integration or the `gh` CLI to resolve the issue and
  open the pull request.
- Work only in the worktree implementation.md describes, and never switch the
  branch of the primary checkout.

## Prerequisites

The issue must already exist. If it does not, file one first with $plan-issue.
Untracked work does not get a pull request.

## Scope

Follow the autonomy mode in `AGENTS.md` and `docs/rules.md`: it decides whether
this skill stops at a draft pull request or goes on to review and merge. The QA
review is not part of this skill; $review-pr is the separate step for it.
