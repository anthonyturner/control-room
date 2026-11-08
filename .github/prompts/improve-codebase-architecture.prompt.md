---
name: 'improve-codebase-architecture'
agent: 'agent'
description: 'Finds shallow modules, ranks where deepening pays most, and ends with a request for planning. Writes nothing. Run only when asked for by name.'
---

Run an architecture review of the area below. Read
`docs/agent-workflows/architecture-review.md` and `docs/design-principles.md`
completely and follow them; they are authoritative. If `architecture-review.md`
is missing, stop and tell the user to update the playbook.

Write nothing: no file, not even a temporary one, no branch, no issue. Run only
read-only `git` commands. If you cannot run `git log`, ask the user to name an
area.

If no area is given below, find hot spots in the history as the workflow says.
