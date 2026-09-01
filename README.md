# answerbank

**A local-first narrative library for grant writers.** Every reusable answer your organizations own, in plain Markdown files on your own disk, with a verification date on each one so you stop shipping last year's staff count.

[![CI](https://github.com/egeria-corporation/answerbank/actions/workflows/ci.yml/badge.svg)](https://github.com/egeria-corporation/answerbank/actions/workflows/ci.yml)

---

## The problem, in your language

Every application asks the same thirty questions. Mission statement. Organizational history. Board composition. Evaluation methodology. DEI statement. Sustainability plan. Logic model. Budget narrative. The wording changes, the word limit changes, the question does not.

So you keep the answers somewhere. If you have one client, that somewhere is a Google Doc called `BOILERPLATE FINAL v3 (use this one).docx`. If you have twelve clients, it is twelve folders, and the honest version of what happens next is this:

- You retype the mission statement into a 250-word box because the file has a 500-word version and nothing in between.
- You paste the board list from March into an application in November, and one of those people resigned in June.
- You report 14 full-time staff because that is what the last successful proposal said, and the organization has had 19 since the spring.
- A funder asks for the same narrative your colleague already wrote for a different proposal, and neither of you can find it.

None of that is a competence problem. It is a filing problem, and it is the single largest recurring time sink in grant work. It is also unglamorous enough that nobody has built the right tool for it.

`answerbank` is the right tool for it. One directory per organization, one Markdown file per answer, word-count variants inside the file, a `last_verified` date on every one, and one command that tells you what has gone stale.

```
$ answerbank stale --all-orgs

OVERDUE
  casa-esperanza  staff-count            verified 2025-09-02   overdue by 63d   (verify every 90d)
  casa-esperanza  board-composition      verified 2025-11-12   overdue by 22d   (verify every 180d)
  riverkeeper     annual-budget          verified 2025-07-01   overdue by 156d  (verify every 180d)

DUE SOON (14 days)
  northside-youth program-outcomes       verified 2026-02-20   due in 6d        (verify every 180d)

42 answers across 4 organizations. 3 overdue, 1 due soon.
```

That command is the point of the whole tool. Everything else is convenience.

---

## Credits

`answerbank` has no upstream dataset, which makes its debts smaller than its sibling projects but not zero.

- **The question taxonomy** in `templates/` encodes conventions developed over decades by working grant professionals and by the common-application efforts that regional grantmaker associations and [Candid](https://candid.org/) have maintained in various forms. Where a structure is traceable to a published form (the federal SF-424 family, the Project Narrative and Budget Narrative attachment forms), the source is cited in [`docs/research/data-sources.md`](docs/research/data-sources.md).
- **[python-docx](https://github.com/python-openxml/python-docx)** by Steve Canny (MIT) does all DOCX generation.
- **[ReportLab](https://www.reportlab.com/opensource/)** (BSD 3-Clause) generates PDF, in the optional `pdf` extra.
- **[PyYAML](https://pyyaml.org/)** (MIT) parses front matter. **[Click](https://click.palletsprojects.com/)** (BSD 3-Clause) is the CLI framework.
- **[Model Context Protocol](https://modelcontextprotocol.io/)** and its Python SDK (MIT) make the agent interface possible.
- **[uv](https://github.com/astral-sh/uv)** and **[ruff](https://github.com/astral-sh/ruff)** by Astral (MIT / Apache 2.0) are the reason the quickstart is one line.

Full attribution with license text pointers is in [`NOTICE`](NOTICE).

---

## 60-second quickstart

No account. No API key. No database. No server.

```bash
# 1. Create a library with one organization, seeded with the 30 standard questions
uvx answerbank init ~/answerbank --org riverkeeper

# 2. Look at what you got
uvx answerbank list --org riverkeeper

# 3. Fill in the mission statement in your own editor
$EDITOR ~/answerbank/riverkeeper/mission.md

# 4. Pull it back out at the length the funder asked for
uvx answerbank get mission --words 250 --org riverkeeper

# 5. The command you will actually run every week
uvx answerbank stale --all-orgs
```

That is the whole loop. Install permanently with `uv tool install answerbank` if you would rather type `answerbank` than `uvx answerbank`.

---

## The file format is the product

An answer is one Markdown file. YAML front matter holds the metadata, `## N words` headings hold the variants, and a `## notes` section holds anything you want to remember but never want exported.

Here is a real one, `~/answerbank/riverkeeper/board-composition.md`:

```markdown
---
id: board-composition
title: Board composition and governance
org: riverkeeper
variants: [100, 300]
last_verified: 2026-08-14
verify_every: 180d
source: >
  Board roster maintained by the Governance Committee. Cross-checked
  against the FY2025 Form 990 Part VII and the minutes of the
  2026-07-09 annual meeting.
tags: [governance, core, required-everywhere]
status: verified
owner: Dana Whitfield
---

## 100 words

Lower Cosumnes Riverkeeper is governed by an eleven-member volunteer
board that meets six times a year. Seven members live within the
watershed the organization serves, and four are members of tribes with
ancestral ties to the lower Cosumnes. The board includes a licensed
hydrologist, a former county floodplain administrator, two small-farm
operators, and a certified public accountant who chairs the Finance
Committee. Officers serve two-year terms with a three-term limit.
Every director completes a conflict-of-interest disclosure annually,
and the full board reviews the executive director's compensation
against comparable regional organizations every other year.

## 300 words

Lower Cosumnes Riverkeeper is governed by an eleven-member volunteer
board of directors...

## notes

The eleven-member figure changed in July 2026 when Marisol Vega
resigned from the seat she had held since 2019. Do not reuse any text
that says "twelve-member" -- three older proposals still contain it.
Sonoran Joint Venture asks for board demographics as a table, not
prose; that lives in board-demographics.md.
```

### Front matter schema

| Field | Required | Type | Notes |
|---|---|---|---|
| `id` | yes | string | Lowercase slug. Must equal the filename without `.md`. Unique within the organization. |
| `title` | yes | string | Human-readable. Shown in `list` and in the MCP tool output. |
| `org` | yes | string | Must equal the directory name the file sits in. A mismatch is an error, not a warning. |
| `variants` | yes | list of integers | The target word counts present in the body. `[100, 300]` means the body has a `## 100 words` section and a `## 300 words` section, and nothing else. |
| `last_verified` | yes | date `YYYY-MM-DD` | The day a human confirmed this text is still true. Not the day the file was edited. |
| `verify_every` | yes | duration | How long this answer stays trustworthy. `30d`, `12w`, `6m`, `1y`. |
| `source` | yes | string or list | Where the facts came from, specifically enough that the next person can re-check them. "Board minutes" is weak. "Minutes of the 2026-07-09 annual meeting" is strong. |
| `tags` | no | list of strings | Free-form. Used by `list --tag` and by search ranking. |
| `status` | no | `draft` \| `verified` \| `retired` | Defaults to `draft`. `retired` answers are excluded from `get` and from all MCP tools unless explicitly requested. |
| `owner` | no | string | Who at the organization is the authority for this answer. Useful when you send the annual verification email. |

Nothing else is permitted in front matter. Unknown keys are an error under `answerbank check`, because a typo'd `last_verifed` that silently does nothing is exactly the failure this tool exists to prevent.

### Why variants live in one file

Because a 150-word mission statement and a 500-word mission statement are the same claim at two lengths, and they go stale together. Splitting them into `mission-150.md` and `mission-500.md` means that when the executive director changes, you have two files to fix and one of them will be missed. That is not a hypothetical. It is the exact class of error this tool exists to prevent.

One file gives you one `last_verified` date governing every length, one git diff that shows the whole change, and one thing to open when a fact moves. The cost is that files get long. A 1,000-word variant plus a 500 plus a 150 is about 1,700 words, roughly 11 KB. That is not a problem worth solving.

Staleness is a property of the fact, not of the phrasing. The format follows the fact.

---

## The `stale` workflow

Run this weekly. Put it in your Monday routine next to checking deadlines.

```bash
answerbank stale --all-orgs
```

It compares `last_verified + verify_every` against today and sorts everything into overdue, due soon, and fine. It exits with status `1` if anything is overdue, so you can wire it into a cron job or a CI check and get an email instead of remembering.

Set `verify_every` to match how fast the underlying fact actually moves. This is the judgment call that makes the tool work:

| Answer | Sensible interval | Why |
|---|---|---|
| Mission statement | `2y` | Changes when the board changes it, which is rare and loud. |
| Organizational history | `1y` | Accretes slowly. Mostly you add a sentence about last year. |
| Evaluation methodology | `1y` | Changes when the program model changes. |
| DEI statement | `1y` | Language conventions in this area move faster than most narrative. |
| Board composition | `180d` | Terms end, people resign, and this is the single most commonly stale answer in the field. |
| Program outcomes and impact numbers | `180d` | Tied to your reporting cycle. |
| Staff count and organizational chart | `90d` | Moves whenever anyone is hired or leaves. |
| Annual budget and audit status | `180d` | Tied to the fiscal year and the audit calendar. |
| Current funders and other support | `90d` | Grants start and end constantly and funders check this one. |

When you have confirmed an answer is still true, stamp it:

```bash
answerbank verify board-composition --org riverkeeper --note "Confirmed with Dana, 11 seats, Vega seat still open"
```

That rewrites `last_verified` to today and appends the note to the `## notes` section with the date. It never touches the answer text.

---

## Working with twelve clients

Consultants do not have one organization. The tool assumes that from the first command.

```
~/answerbank/
├── answerbank.toml              # library config: default org, export defaults
├── riverkeeper/
│   ├── _org.toml                # display name, EIN, fiscal year end, notes
│   ├── mission.md
│   ├── organizational-history.md
│   ├── board-composition.md
│   └── ...
├── casa-esperanza/
│   ├── _org.toml
│   └── ...
└── northside-youth/
    └── ...
```

```bash
answerbank orgs                                  # list every organization
answerbank get mission --org casa-esperanza      # explicit
answerbank get mission                           # uses default_org from answerbank.toml
answerbank stale --org casa-esperanza            # one client
answerbank stale --all-orgs                      # Monday morning
```

Set the default once and stop typing `--org` for the client you are in this week:

```toml
# ~/answerbank/answerbank.toml
default_org = "riverkeeper"
```

Or override per shell session with `ANSWERBANK_DEFAULT_ORG=casa-esperanza`.

Because a library is a directory of Markdown files, `git init` inside it gets you a full history of every narrative change for every client, for free. When an engagement ends, `git log riverkeeper/` is the handoff document, and `cp -r riverkeeper/` is the handoff.

---

## Export, with the counts funders actually enforce

```bash
answerbank export mission --words 500 --org riverkeeper --format docx --out mission.docx
answerbank export mission evaluation sustainability --words 500 --format pdf --out packet.pdf
answerbank get mission --words 150 --copy          # straight to the clipboard
```

Every export reports what you are actually about to paste:

```
$ answerbank export need-statement --words 500 --org casa-esperanza --limit-words 500 --format docx --out need.docx

Wrote need.docx
  words              517
  characters         3,344
  characters (no spaces)  2,861

  WARNING: 517 words exceeds the stated limit of 500 by 17 words.
```

Word and character limits are hard gates in most funder portals, and the portal usually truncates silently rather than warning you. `--limit-words` and `--limit-chars` make the tool warn instead. Exports exceeding a stated limit still write the file and still exit non-zero, so you notice.

The counting rules are documented exactly in [`docs/research/data-sources.md`](docs/research/data-sources.md), because Microsoft Word, Google Docs, and Submittable do not agree with each other and you need to know which one this tool imitates.

---

## Using this with Claude

This is the part that changes the work rather than speeding it up.

`answerbank` ships an MCP (Model Context Protocol) server. Pointed at your library, Claude can draft a complete application for a specific funding opportunity using **only** your client's verified, current language. Not generic AI prose about community impact. Your organization's own approved sentences, assembled to fit the funder's questions and word limits, with the verification date attached to every source it used.

Funders have gotten fast at spotting generated proposals, and the reaction has been hostile. The defensible version of AI in this workflow is not "write me a grant." It is "assemble the organization's verified narrative against this funder's questions, and tell me which claims are stale." That is what this server does.

### Claude Desktop

Edit `claude_desktop_config.json` (macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`; Windows: `%APPDATA%\Claude\claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "answerbank": {
      "command": "uvx",
      "args": ["answerbank", "mcp", "--library", "/Users/dana/answerbank"]
    }
  }
}
```

Restart Claude Desktop. The tools appear under the connector menu.

### Claude Code

```bash
claude mcp add answerbank -- uvx answerbank mcp --library ~/answerbank
```

### What the server exposes

Read-only by default. The server cannot modify your files, and cannot reach the network at all, unless you start it with `--allow-write` or `--allow-network`.

| Tool | What it does |
|---|---|
| `answerbank_list_orgs` | The organizations in the library. |
| `answerbank_list_answers` | Every answer for an organization, with title, tags, variants, and staleness. |
| `answerbank_get_answer` | One answer at a requested length, with its verification date and source. |
| `answerbank_search` | Keyword search across titles, tags, and body text. Local, deterministic, no embeddings. |
| `answerbank_stale` | What is overdue for verification. |
| `answerbank_draft_context` | The heart of it. Given a funder's question, returns the best-matching verified answers assembled as grounding material, with an explicit instruction that these are the only permitted sources and a list of anything stale that the draft should not rely on. |
| `answerbank_check_limits` | Counts words and characters in a draft against a stated funder limit. |

A session that works looks like this:

> Here is the NOFO for the EPA Environmental Justice Government-to-Government program. Using only Lower Cosumnes Riverkeeper's answer bank, draft the Community Engagement and Statement of Need sections. Respect the 1,500-word limit on each. Flag anything you had to use that is past its verification date, and tell me what is missing entirely.

The last clause is the one that earns its keep. The gaps in an answer bank are the gaps in the proposal, and finding them two weeks before the deadline is worth more than the drafting.

---

## Privacy: a promise, not a setting

This tool holds your clients' narratives on your disk. Some of it is unpublished. Some of it is sensitive: program participant descriptions, financial detail, internal notes about a board conflict.

**Nothing leaves your machine unless you explicitly ask it to.**

Concretely, and enforced in the code rather than in the documentation:

- There is no telemetry, no analytics, no crash reporting, and no update check. The tool makes zero network calls on its own initiative, ever.
- The **only** commands that touch the network are `answerbank pull` and `answerbank push`, they only run when you type them, and they do nothing at all unless you have set an OpenGrants API key yourself.
- The MCP server has no network capability. It reads files and returns text. Starting it with `--allow-network` is the only way to change that, and it is off by default.
- The MCP server is read-only unless started with `--allow-write`.
- Your API key, if you set one, is never written to disk by this tool, never logged, and never included in output or error messages.
- Nothing is written outside the library directory you named.

If you find a code path that violates any of the above, that is a security bug, and [`SECURITY.md`](SECURITY.md) tells you how to report it.

Using the MCP server does send the answers Claude requests to Anthropic, the same as pasting them into a chat window. That is your explicit action, and it is worth being deliberate about which client libraries you point a model at.

---

## Optional: OpenGrants

`answerbank` is fully functional with no credentials at all, and this is the only place in the documentation that mentions otherwise.

If you have an [OpenGrants](https://opengrants.io) API key in `OPENGRANTS_API_KEY`, two extra commands work:

- `answerbank pull --org riverkeeper` pre-seeds a new library from the organization profile OpenGrants already holds, so you start with the EIN, mission, service area, and budget filled in instead of an empty template.
- `answerbank push --org riverkeeper` sends the completed narrative set back as a richer matching profile, which materially improves opportunity matching. `--dry-run` shows you exactly what would be sent, field by field, before anything moves.

Both degrade silently. No key, no network, expired key, rate limit, anything: the rest of the tool works exactly as it did. Output sourced from the API is marked `— live from OpenGrants` so you always know which lines came from your files and which came from the network. Keys go in `.env` or your shell environment, never in the repo. See [`.env.example`](.env.example).

---

## What this tool does not claim

`answerbank` stores what you tell it and reports the dates you set. A `last_verified` date is your assertion that you checked, not a verification performed by this software. `answerbank stale` tells you when you said you would check again. It cannot tell you whether an answer is true.

The program-wide disclosure about public-data-derived claims does not apply to your own narrative files, because none of it is derived from public data. It does apply to anything returned by `answerbank pull`:

> This is informational only, derived from public data on the dates shown. It is not an eligibility determination, and not legal, tax, or accounting advice. Verify against the official source before relying on it.

What the tool will never do is in [`docs/NON-GOALS.md`](docs/NON-GOALS.md). Read it before filing a feature request.

---

## Documentation

- [`docs/NON-GOALS.md`](docs/NON-GOALS.md) — what this is not, and will not become
- [`docs/research/data-sources.md`](docs/research/data-sources.md) — the thirty-question taxonomy, federal and foundation application structures, and how word and character limits actually get counted
- [`docs/research/prior-art.md`](docs/research/prior-art.md) — what people use today and why it does not fit
- [`docs/research/competitive.md`](docs/research/competitive.md) — the paid features this replaces
- [`docs/hosted/architecture.md`](docs/hosted/architecture.md) — the template gallery at answers.opengrants.io
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — including how to contribute a template answer

The template gallery at **[answers.opengrants.io](https://answers.opengrants.io)** is the same thirty questions with guidance on what a strong answer contains, readable without installing anything. If you are early in a development career, working through it in order is a serviceable curriculum in what a complete organizational narrative looks like. The answer bank you build is portable. It goes with you to the next job.

---

## License

Apache License 2.0. See [`LICENSE`](LICENSE) and [`NOTICE`](NOTICE).

The starter templates are covered by the same license. Using that text as the starting point for a real grant application is not distribution and asks nothing of you. Attribution is only required if you redistribute the templates themselves.

---

Built and maintained by Egeria Corporation, sponsored by [OpenGrants](https://opengrants.io).
