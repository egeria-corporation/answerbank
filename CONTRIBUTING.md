# Contributing to answerbank

Two very different kinds of contribution matter here, and the second one is not code.

1. **Code and bug fixes.** Standard open-source workflow, described below.
2. **Template answers for the starter gallery.** If you are a grant professional and you have never sent a pull request, this is where you belong, and the barrier is deliberately low. Skip to [Contributing a template answer](#contributing-a-template-answer).

---

## Before you start

Read [`docs/NON-GOALS.md`](docs/NON-GOALS.md). It is short, and it will save you the disappointment of building something well that we then decline. The list is not a stalling tactic; it is the reason the tool stays usable.

The two rules that govern every decision in this repo:

- **The quickstart stays one command.** No account, no key, no database, no server. If a change makes `uvx answerbank init` require something new, it is the wrong change.
- **Nothing leaves the user's machine without an explicit action.** This is not a preference. A pull request that adds telemetry, an update check, a crash reporter, a "help us improve" prompt, or any network call outside `pull` and `push` will be closed. See the privacy promise in the README.

---

## Development setup

```bash
git clone https://github.com/egeria-corporation/answerbank
cd answerbank
uv sync --all-extras --dev

uv run pytest -q
uv run ruff check .
uv run ruff format .
uv run answerbank --help
```

Python 3.11 or newer. CI runs 3.11, 3.12, and 3.13, so do not use syntax newer than 3.11 supports.

To try your changes against a scratch library without touching your real one:

```bash
uv run answerbank init /tmp/scratch-library --org test-org
uv run answerbank --library /tmp/scratch-library list --org test-org
```

---

## Code contributions

### Where code goes

The architecture has one rule and it is enforced in review: **business logic lives in the library modules, never in a CLI command handler or an MCP tool handler.** Both of those are thin adapters over `answerbank.library` and friends. If a behavior is available through the CLI but not through MCP, that is a bug, and the fix is almost always to move the logic down a layer.

```
src/answerbank/
├── model.py         dataclasses only, no I/O
├── frontmatter.py   parse and serialize the file format
├── library.py       discovery, resolution, load, save  <- the core API
├── counts.py        word and character counting, limit checks
├── staleness.py     duration parsing and due-date math
├── search.py        deterministic local ranking
├── templates.py     the bundled starter set
├── export/          one module per output format
├── opengrants.py    the only module allowed to open a socket
├── cli.py           thin
└── mcp_server.py    thin
```

### Dependencies

Adding a runtime dependency requires a conversation in an issue first. The dependency list is short on purpose: `uvx answerbank` has to start fast, and every package added is a package that can break a Monday morning. Anything requiring a compiler or a system library (Cairo, Pango, a headless browser, pandoc) is out.

### Tests

Every change ships with a test. Tests use real fixture files in `tests/fixtures/`, meaning actual Markdown answer files with real front matter, not mocks of what we think the format looks like. Format drift is the failure mode that matters, and only real fixtures catch it.

Specific things a test must cover if you touch them:

- Front matter round-trips. Parse a file, serialize it, and the bytes should be identical apart from the field you deliberately changed. `verify` must not reflow the user's YAML or reorder their keys.
- Word counting is exact. If you change `counts.py`, add a fixture with a known count computed by hand and asserted literally.
- Staleness math across month and year boundaries, including February and leap years.
- Every CLI command has a test that runs it through Click's `CliRunner` and asserts on both stdout and the exit code.

### Pull requests

- One concern per pull request.
- Ruff clean, tests green, before you open it.
- Update the README if you changed a command's behavior. A command whose documentation is wrong is worse than a missing command.
- Add a `CHANGELOG.md` entry under `## Unreleased`.
- Describe the user-visible behavior in the PR body, in the words a grant consultant would use. "Adds `--strict` to `check`" is not a description. "`answerbank check --strict` now fails when an answer's body has a heading that is not declared in `variants`, which is how a stray `## 250 words` section goes unnoticed" is.

---

## Contributing a template answer

The starter gallery is the thirty standard questions, each with a skeleton answer and guidance on what a strong answer contains. It ships inside the package (so `answerbank init` seeds a real library) and it is published as the public gallery at [answers.opengrants.io](https://answers.opengrants.io). Both come from the same files in `templates/`.

This is the most valuable contribution a practicing grant professional can make, and it is the reason a beginner can install this tool and learn something rather than staring at empty files.

### What a template file looks like

Templates live in `templates/<question-id>.md`. Same format as any answer, plus a `guidance` block. Templates carry no organization and no real facts.

```markdown
---
id: sustainability-plan
title: Sustainability plan
org: _template
variants: [150, 400]
last_verified: 2026-08-30
verify_every: 1y
source: TEMPLATE -- replace with your own source before using
tags: [finance, core, required-everywhere]
status: draft
---

## guidance

**What the funder is actually asking.** Not "will you survive," but
"what happens to this specific program when this specific money ends."
Reviewers have read a thousand answers that say "we will pursue
diversified funding streams" and they discount all of them.

**What a strong answer contains.**

1. A named plan for the program after the grant period, not for the
   organization in general.
2. Specific successor funding with names and stages: which funders,
   applied or planned, what amounts, what dates.
3. Evidence you have done this before. A prior program that outlived
   its launch grant is the single most persuasive sentence available
   to you.
4. What gets cut if the money does not come. Reviewers trust an
   applicant who says "the two outreach positions would not continue
   and we would serve 40 percent fewer families" far more than one who
   claims everything continues.
5. Non-cash sustainability where it is real: earned revenue, a formal
   partner commitment, a policy change that outlasts the funding.

**Common mistakes.** Naming funders you have never contacted.
Confusing organizational sustainability with program sustainability.
Promising a fundraising event that does not exist yet. Writing this
section last, at 11pm, in the voice of somebody who has given up.

**Word limits seen in the field.** Federal narratives commonly allot
one to two pages. Foundation applications commonly ask for 150 to 500
words. Community foundation LOIs often ask for two sentences. Keep a
150 and a 400 and you will cover most of it.

## 150 words

[Program name] will continue after the [funder] grant period through
[specific mechanism]. In [year], [organization] sustained [prior
program] beyond its initial [funder] award by [what actually
happened], and [prior program] now operates on [current funding
source]...

## 400 words

...
```

The `## guidance` section is stripped out when the template is copied into a real library, so a consultant gets a clean file. It is what the public gallery publishes.

### What makes a good template contribution

- **Guidance drawn from reading real reviews.** If you have served on a review panel, say what actually loses points. That is the content nobody else can write.
- **Skeleton text with bracketed slots**, not finished prose. A template that reads as a complete answer will get pasted in unchanged by somebody in a hurry, and that is how identical proposals land on the same program officer's desk. Bracketed slots make the copying visible.
- **No real organization's language.** Do not contribute your client's mission statement, even a good one, and even with permission. Templates are structural.
- **No jurisdiction-specific claims presented as universal.** If something is true of federal applications but not foundations, say which.
- **Cite a form or a published instruction where one exists**, so `docs/research/data-sources.md` can carry the reference.

### How to submit one

If you are comfortable with git: fork, add the file to `templates/`, run `uv run pytest tests/test_templates.py` (it validates front matter and checks that every `variants` entry has a matching body section), and open a pull request.

If you are not comfortable with git: open an issue using the **Template answer** issue template and paste the text in. A maintainer will format and commit it with you credited in the pull request. This is a completely normal way to contribute here and nobody will think less of you for it.

### Attribution

Contributors of template content are listed in `templates/CREDITS.md` with a link of their choosing. If your firm's name belongs there rather than yours, say so and it will be.

Template contributions are licensed Apache 2.0 along with the rest of the repository. Using a template as the starting point for a real grant application is not distribution and requires nothing of you.

---

## Reporting bugs

Include the answerbank version (`answerbank --version`), your Python version, your operating system, the exact command, and what happened. If it involves a specific answer file, include a redacted version of the front matter. Never paste a client's narrative into a public issue.

Security issues go to security@egeriacorp.com, not to the issue tracker. See [`SECURITY.md`](SECURITY.md).

---

## Code of conduct

[`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md) applies everywhere in this project. The short version: many people who will use this tool are underpaid, overworked, and doing the work anyway. Extend them and each other the patience you would want in your worst week.
