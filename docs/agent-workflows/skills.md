# Skill map

A **skill** is a named entry point an agent tool exposes — a command the
requester types to run one piece of the workflow. This file lists every one,
the canonical document it loads, and how each tool invokes it. It also lists
the agent behind each pipeline stage.

It exists so that a gap is visible rather than accidental. When one tool has a
skill and another does not, this map records whether that is a decision or an
oversight.

## The rule

A skill file holds **platform metadata, capability restrictions, and a pointer to
the canonical workflow. Nothing else.** This is the same rule agents follow, from
**Platform adapters** in [pipeline.md](pipeline.md).

A skill that restates a stage rule becomes a second definition of that stage, and
two definitions drift apart silently — one gets corrected and the other does not.
When a skill needs behavior that is not written down yet, the behavior goes in
`docs/agent-workflows/` first and the skill points at it.

## Pipeline skills

These are the same on every tool: same names, same scope, same document
behind them.

| Skill | Canonical document | Takes | Returns |
| --- | --- | --- | --- |
| `plan-issue` | [planning.md](planning.md) | A request | A filed issue |
| `implement-issue` | [implementation.md](implementation.md) | An issue number | A reviewed, merged pull request |
| `review-pr` | [qa-review.md](qa-review.md) | A pull request | A review posted on the pull request |
| `open-pr` | [implementation.md](implementation.md) | Uncommitted work plus an issue | A reviewed, merged pull request |
| `ship-feature` | [pipeline.md](pipeline.md) | A request | A reviewed, merged pull request |
| `grill-me` | [grilling.md](grilling.md) | A plan or idea | Shared understanding |
| `grill-with-docs` | [domain-modeling.md](domain-modeling.md) | A plan or idea | Shared understanding, plus a glossary |
| `wayfinder` | [wayfinding.md](wayfinding.md) | A loose idea too big for one session | A map of decision tickets on GitHub |
| `improve-codebase-architecture` | [architecture-review.md](architecture-review.md) | A codebase, or an area of it | A request for `plan-issue`, in the reply |

They chain in the order their names describe:

```text
plan-issue  ──▶  implement-issue  ──▶  review-pr
PM + refine + UX    engineer → PR        QA → verdict
     │                     ▲
     └──── issue #n ───────┘        open-pr ──┘
                               (recovery: code written first)

ship-feature = all three, end to end
```

`open-pr` is the recovery path, not a fourth stage. It reaches the same pull
request as `implement-issue`, entering the workflow after the code was already
written — see **When the work was written before the pull request existed** in
[implementation.md](implementation.md).

`grill-me` produces no artifact and files nothing. `grill-with-docs` is the same
session plus a glossary, and may write that glossary and, once the requester
agrees, a decision record; it writes nothing else and files nothing
([domain-modeling.md](domain-modeling.md)). `wayfinder` plans work too big for
one session as a map of decision tickets; it writes only GitHub issues and
comments, never a repository file, branch or pull request, and it hands over to
`plan-issue` rather than sitting in front of it ([wayfinding.md](wayfinding.md)).
`improve-codebase-architecture` looks across the codebase for shallow modules,
grills the one candidate the requester picks, and ends with a request for
`plan-issue`; it writes no file at all and files nothing
([architecture-review.md](architecture-review.md)). All four are opt-in, by name
only ([rule 13](../rules.md)). Their metadata turns off automatic invocation
wherever the tool supports that, so there the loader enforces the rule and prose
does not have to. Copilot is the exception, noted under its table below.

## How each tool invokes them

**Claude Code.** The skills and agents ship in the agent-playbook plugin, so
they are namespaced: type `/agent-playbook:<skill>`, for example
`/agent-playbook:plan-issue`. The stage agents are `agent-playbook:pm`,
`agent-playbook:refine`, `agent-playbook:ux-design`, `agent-playbook:dev` and
`agent-playbook:qa`; the skills start them as subagents.

**Codex.** Type `$<skill>`, for example `$plan-issue`. The skills live in
`.agents/skills/<skill>/SKILL.md`, and the stage agents in
`.codex/agents/<name>.toml`. The opt-in skills (`grill-me`, `grill-with-docs`,
`wayfinder` and `improve-codebase-architecture`) each carry `agents/openai.yaml` with
`allow_implicit_invocation: false`, so Codex cannot start them unasked.

**GitHub Copilot.** Copilot has no skill command here; pick the custom agent
from the agent picker in Copilot Chat instead. The agents live in
`.github/agents/<name>.agent.md`.

| Skill | Copilot custom agent |
| --- | --- |
| `plan-issue` | `orchestrator`, asked to plan only |
| `implement-issue` | `software-engineer` |
| `open-pr` | `software-engineer`, pointed at the uncommitted work |
| `review-pr` | `qa-reviewer` |
| `ship-feature` | `orchestrator` |
| `grill-me` | `product-manager`, asked for a grilling session by name |
| `grill-with-docs` | Built-in agent mode through the prompt `grill-with-docs` (see below) |
| `wayfinder` | Built-in agent mode through the prompt `wayfinder` (see below) |
| `improve-codebase-architecture` | Built-in agent mode through the prompt `improve-codebase-architecture` (see below) |

The `orchestrator` routes a request through every stage in order — product
manager, refinement, optional UX design, software engineer, QA reviewer — and
never skips refinement on substantial work. `instructions-generator` is not a
pipeline stage: it drafts new instruction files, run through
`.github/prompts/create-instructions.prompt.md`.

`grill-with-docs` writes files, which `product-manager` cannot, so it is a
prompt file that runs in Copilot's built-in agent mode,
`.github/prompts/grill-with-docs.prompt.md`.

`wayfinder` is a prompt file too, `.github/prompts/wayfinder.prompt.md`,
because `product-manager` is bound to the issue format in
[product-manager.md](product-manager.md), and that format does not apply to maps
or decision tickets ([tracking.md](tracking.md)). The trade-off is that
built-in agent mode can edit files and run commands, `git push` among them, and
this playbook sets no tool restriction on it. So `wayfinder`'s limits (GitHub
issues and comments only, nothing written to the repository) rest on the
prompt's wording and on [wayfinding.md](wayfinding.md), not on the tool.

`improve-codebase-architecture` needs a terminal to read the history's hot spots,
which the read-only `refinement` agent lacks and no custom agent here can limit
to read-only `git`. It is therefore a prompt file too,
`.github/prompts/improve-codebase-architecture.prompt.md`, which runs in the
built-in agent mode with a terminal. Built-in agent mode can also edit files, so
"writes nothing" rests on the prompt and on the workflow, not on the tool.
Without a terminal, the review asks the requester to name an area.

A prompt runs only when typed by name, but Copilot has no setting that stops it
being used automatically, as Claude Code and Codex have. For all three prompts
(`grill-with-docs`, `wayfinder` and `improve-codebase-architecture`) the opt-in
therefore rests on being typed by name and on [rule 13](../rules.md), not on the
loader. Treat that as a known gap.

## Stage agents

| Stage | Canonical document | Claude Code |
| --- | --- | --- |
| Product Manager | [product-manager.md](product-manager.md) | `agent-playbook:pm` |
| Engineering Refinement | [refinement.md](refinement.md) | `agent-playbook:refine` |
| UX Design | [ux-design.md](ux-design.md) | `agent-playbook:ux-design` |
| Software Engineer | [implementation.md](implementation.md) | `agent-playbook:dev` |
| QA Reviewer | [qa-review.md](qa-review.md) | `agent-playbook:qa` |

In Codex the same stages are the agents `pm`, `refinement`, `ux_design`,
`developer` and `qa`. `refinement` and `ux_design` run read-only without
network; `pm`, `developer` and `qa` run in the session's own sandbox, so they
reach GitHub only when the session allows network access.
In Copilot the same stages are the custom agents `product-manager`,
`refinement`, `ux-design`, `software-engineer` and `qa-reviewer`.

## Plugin commands

Two commands ship with the plugin and are not pipeline stages.
`/agent-playbook:init` installs or updates this playbook in a project.
`/agent-playbook:smell` runs a read-only code-smell review and changes nothing.
It gives line-level findings on the branch diff, before merge. The
`improve-codebase-architecture` skill is its module-level counterpart: it reads
the history's hot spots instead of a diff, and what it finds becomes the next
refactor's request. `smell` is Claude Code only; the skill is on every tool.

## Project task skills

Procedures for specific, recurring jobs in this repository go in
`.claude/skills/<name>/SKILL.md`. Unlike the pipeline skills these are
self-contained: the procedure is the skill, and there is no canonical document
to point at because nothing else needs it. List each one here with what it
does and which tools carry it, so a missing copy on one tool is a recorded gap
rather than an accident.

| Skill | What it does | Tools |
| --- | --- | --- |
| _none yet_ | | |

## Naming

**`<verb>-<object>`, where the object is the artifact you end up holding.**

`implement-issue` says what you get. A name like `run-pipeline` says only that
a machine ran — not what it produces, and not where it stops. That vagueness is
how one name ends up meaning two different scopes on two tools.

Avoid two names that differ only by a suffix. They do different things and get
picked wrongly under time pressure; a name that is visibly distinct is worth
more than consistency with a convention.

**One exception, recorded once:** skills adopted from
[mattpocock/skills](https://github.com/mattpocock/skills) keep their upstream
names, as `grill-me` did, so people who know the upstream skill find it. The
licence notice is in
[third-party-notices.md](../third-party-notices.md).
`grill-with-docs` is the suffix case: it is `grill-me` plus a glossary, and it
is named for what it adds. `wayfinder` is the other kind: it is not a
`<verb>-<object>` name, and it keeps the upstream name under this exception.
`improve-codebase-architecture` is the third kind: it is a `<verb>-<object>`
name that promises more than the skill does, since the skill only proposes and
the improvement is a later `refactor` issue. It keeps the upstream name too.

## Adding a platform

1. Create the platform's skill directory.
2. For each row in **Pipeline skills**, add an adapter under the same name that
   loads the canonical document.
3. Add the platform to **How each tool invokes them** and **Stage agents**.
4. Add the adapter location to **Platform adapters** in
   [pipeline.md](pipeline.md).

No workflow content is written or copied at any step. If a platform genuinely
cannot support a skill, say so here and say why.
