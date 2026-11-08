---
name: improve-codebase-architecture
description: Finds shallow modules across control-room, ranks where deepening pays most, grills the one the user picks, and ends with a request for plan-issue. Changes no code and writes no file. Use ONLY when the user invokes $improve-codebase-architecture or asks for it by name. Never start it on your own initiative, and never as a precondition for filing an issue.
---

# Review the architecture and propose a refactor (Codex)

Read `docs/agent-workflows/architecture-review.md` and
`docs/design-principles.md` completely and run the review they define. They are
authoritative; this file adds only what is specific to Codex. If
`architecture-review.md` is missing, stop and tell the user to update the
playbook. The technique is adapted from
[mattpocock/skills](https://github.com/mattpocock/skills) (MIT).

## Codex specifics

- Run the review in this conversation with the user. Hand the exploration to a
  read-only subagent if you like; the grilling never goes to a subagent,
  because it cannot interview a person.
- The hot-spot step reads history with `git log`. If the sandbox refuses it,
  ask the user to name an area instead.

## Scope

Write no file, not even a temporary one, create no branch, and file no issue as
part of this skill. The review ends with a request for $plan-issue, which is the
user's call and not a step of this skill.
