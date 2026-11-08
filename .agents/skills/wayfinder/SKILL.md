---
name: wayfinder
description: Plans work too big for one session as a map of decision tickets on control-room's GitHub and settles them one at a time. Use ONLY when the user invokes $wayfinder or asks for it by name. Never start it on your own initiative, and never as a precondition for filing an issue.
---

# Map a large effort as decision tickets (Codex)

Read `docs/agent-workflows/wayfinding.md` and the **Wayfinding** section of
`docs/agent-workflows/tracking.md` completely and run the session they define.
They are authoritative; this file adds only what is specific to Codex. If
`wayfinding.md` is missing, or `tracking.md` has no **Wayfinding** section, stop
and tell the user to update the playbook. The
technique is adapted from [mattpocock/skills](https://github.com/mattpocock/skills)
(MIT).

## Codex specifics

- Run the session in this conversation with the user. A subagent cannot
  interview a person, so delegate only research tickets, which
  `wayfinding.md` allows. The `refinement` agent has no network, so research
  that needs the web runs in this session.

## Scope

Write no repository file, create no branch, and open no pull request as part of
this skill. A prototype's throwaway work, kept outside the repository's branches,
follows `wayfinding.md`. A map ends by handing over to $plan-issue, which is the user's call
and not a step of this skill.
