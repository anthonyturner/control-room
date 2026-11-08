---
name: 'wayfinder'
agent: 'agent'
description: 'Plans work too big for one session as a map of decision tickets on GitHub. Run only when asked for by name.'
---

Run a wayfinding session for the request below. Read
`docs/agent-workflows/wayfinding.md` and the **Wayfinding** section of
`docs/agent-workflows/tracking.md` completely and follow them; they are
authoritative. If `wayfinding.md` is missing, or `tracking.md` has no **Wayfinding**
section, stop and tell the user to update the playbook.

Write nothing to the repository, create no branch, and open no pull request. A
prototype's throwaway work, kept outside the repository's branches, follows
`wayfinding.md`.

If no request or map is given below, ask what the user wants to map.
