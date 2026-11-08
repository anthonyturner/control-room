---
name: grill-with-docs
description: A grilling session that also keeps the control-room glossary and offers decision records as choices settle. Use ONLY when the user invokes $grill-with-docs or asks for it by name. Never start a session on your own initiative, and never as a precondition for filing an issue.
---

# Grill a request and write down its language (Codex)

Read `docs/agent-workflows/grilling.md` and
`docs/agent-workflows/domain-modeling.md` completely and run the session they
define. They are authoritative; this file adds only what is specific to Codex.
If `domain-modeling.md` is missing, stop and tell the user to update the
playbook. The technique is adapted from
[mattpocock/skills](https://github.com/mattpocock/skills) (MIT).

## Codex specifics

- Run the session in this conversation with the user. A subagent cannot
  interview a person, so never delegate it.
- Write only what `domain-modeling.md` allows, and follow it for what to do
  when the sandbox will not let you write.

## Scope

The files, branch and issue limits are in `domain-modeling.md`. If the user then
wants an issue filed, hand the settled decisions to $plan-issue. That is their
call, not a step of this skill.
