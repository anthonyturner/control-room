# Third-party notices

Some of the workflows in `docs/agent-workflows/`, the testing guide in
`docs/testing.md`, and the design vocabulary in `docs/design-principles.md`,
draw on techniques from the project below. This
file travels with them into every project the playbook is installed in; the notice that follows is reproduced as its licence requires, so
keep it.

## mattpocock/skills

<https://github.com/mattpocock/skills>, MIT licence. Adapted from the state of
the repository at commit `49dd158d1076134a641b33efb035946536778336`.

| Installed file | Adapts |
| --- | --- |
| `docs/agent-workflows/grilling.md` | `skills/productivity/grilling` |
| `docs/agent-workflows/domain-modeling.md` | `skills/engineering/grill-with-docs` and `skills/engineering/domain-modeling` (including `GLOSSARY-FORMAT.md` and `ADR-FORMAT.md`) |
| `docs/agent-workflows/wayfinding.md` | `skills/engineering/wayfinder` (including its `agents/openai.yaml`) |
| `docs/agent-workflows/architecture-review.md` | `skills/engineering/improve-codebase-architecture` (including `HTML-REPORT.md` and `agents/openai.yaml`) and the vocabulary and principles of `skills/engineering/codebase-design` (including `DEEPENING.md` and `DESIGN-IT-TWICE.md`) |
| `docs/testing.md` | `skills/engineering/tdd` (including `mocking.md` and `tests.md`) |
| `docs/design-principles.md` | The ideas behind the design vocabulary, from `skills/engineering/codebase-design`; the text is the playbook's own |

The playbook restates these techniques in its own words and adds its own
project addendum. Skill names such as `grill-me`, `grill-with-docs`, `wayfinder`
and `improve-codebase-architecture` follow the upstream names.

```text
MIT License

Copyright (c) 2026 Matt Pocock

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
